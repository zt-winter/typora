# TCP重组

[开源项目中相关代码](github.com/asmcos/sniffer/blob/master/sniffer.go#1056)

* gopacket中关于TCP重组有两个库，一个是tcpassembly，另一个是reassembly。tcpassembly功能不完善，bug比较多，并且已经很久没有维护。reassembly非常完善，因此在TCP重组实现时选择使用reassembly

* 一般实现TCP重组过程中基本变量初始化为steamFacotry -> streamPool -> assembler -> AssembleWithContext -> FlushWithOptions

* AssembleWithContext在函数执行过程调用三个重要的函数StreamFactory.New(Creating a stream)	ReassembledSG(on a single stream)	ReassemblyComplete(on the same stream)

* 其中StreamFactory.New函数，由接口StreamFactory定义,AssembleWithContext函数调用getConnection函数，getConnection函数会调用StreamFactory.New()。StreamFactory接口定义New方法，需要使用者自己去构造New函数，新建的stream就可以在New函数中进行操作。[getConnection函数位置](https://github.com/google/gopacket/blob/v1.1.19/reassembly/memory.go#233)

* ReassembledSG函数，在接口Stream中定义，AssembleWithContext函数调用sendToConnection函数，再调用ReassembledSG。同样需要使用者自己去构造ReassembledSG函数。[sendToConnection函数位置](https://github.com/google/gopacket/blob/v1.1.19/reassembly/tcpassembly.go#1101)

* ReassemblyComplete函数，在接口Stream中定义，AssembleWithContext函数调用sendToConnection函数，再调用closeHalfConnection函数，再调用ReassemblyComplete函数。[closeHalfConnection函数位置](https://github.com/google/gopacket/blob/v1.1.19/reassembly/tcpassembly.go#1198)

* 在接口stream中还定义了Accept函数，由该函数可以实现如何选择正确的TCP数据包加入到流中


```
// AssembleWithContext reassembles the given TCP packet into its appropriate
// stream.
//
// The timestamp passed in must be the timestamp the packet was seen.
// For packets read off the wire, time.Now() should be fine.  For packets read
// from PCAP files, CaptureInfo.Timestamp should be passed in.  This timestamp
// will affect which streams are flushed by a call to FlushCloseOlderThan.
//
// Each AssembleWithContext call results in, in order:
//
//    zero or one call to StreamFactory.New, creating a stream
//    zero or one call to ReassembledSG on a single stream
//    zero or one call to ReassemblyComplete on the same stream
func (a *Assembler) AssembleWithContext(netFlow gopacket.Flow, t *layers.TCP, ac AssemblerContext) {
	var conn *connection
	var half *halfconnection
	var rev *halfconnection

	a.ret = a.ret[:0]
	key := key{netFlow, t.TransportFlow()}
	ci := ac.GetCaptureInfo()
	timestamp := ci.Timestamp

	conn, half, rev = a.connPool.getConnection(key, false, timestamp, t, ac)
	if conn == nil {
		if *debugLog {
			log.Printf("%v got empty packet on otherwise empty connection", key)
		}
		return
	}
	conn.mu.Lock()
	defer conn.mu.Unlock()
	if half.lastSeen.Before(timestamp) {
		half.lastSeen = timestamp
	}
	a.start = half.nextSeq == invalidSequence && t.SYN
	if *debugLog {
		if half.nextSeq < rev.ackSeq {
			log.Printf("Delay detected on %v, data is acked but not assembled yet (acked %v, nextSeq %v)", key, rev.ackSeq, half.nextSeq)
		}
	}

	if !half.stream.Accept(t, ci, half.dir, half.nextSeq, &a.start, ac) {
		if *debugLog {
			log.Printf("Ignoring packet")
		}
		return
	}
	if half.closed {
		// this way is closed
		if *debugLog {
			log.Printf("%v got packet on closed half", key)
		}
		return
	}

	seq, ack, bytes := Sequence(t.Seq), Sequence(t.Ack), t.Payload
	if t.ACK {
		half.ackSeq = ack
	}
	// TODO: push when Ack is seen ??
	action := assemblerAction{
		nextSeq: Sequence(invalidSequence),
		queue:   true,
	}
	a.dump("AssembleWithContext()", half)
	if half.nextSeq == invalidSequence {
		if t.SYN {
			if *debugLog {
				log.Printf("%v saw first SYN packet, returning immediately, seq=%v", key, seq)
			}
			seq = seq.Add(1)
			half.nextSeq = seq
			action.queue = false
		} else if a.start {
			if *debugLog {
				log.Printf("%v start forced", key)
			}
			half.nextSeq = seq
			action.queue = false
		} else {
			if *debugLog {
				log.Printf("%v waiting for start, storing into connection", key)
			}
		}
	} else {
		diff := half.nextSeq.Difference(seq)
		if diff > 0 {
			if *debugLog {
				log.Printf("%v gap in sequence numbers (%v, %v) diff %v, storing into connection", key, half.nextSeq, seq, diff)
			}
		} else {
			if *debugLog {
				log.Printf("%v found contiguous data (%v, %v), returning immediately: len:%d", key, seq, half.nextSeq, len(bytes))
			}
			action.queue = false
		}
	}

	action = a.handleBytes(bytes, seq, half, t.SYN, t.RST || t.FIN, action, ac)
	if len(a.ret) > 0 {
		action.nextSeq = a.sendToConnection(conn, half, ac)
	}
	if action.nextSeq != invalidSequence {
		half.nextSeq = action.nextSeq
		if t.FIN {
			half.nextSeq = half.nextSeq.Add(1)
		}
	}
	if *debugLog {
		log.Printf("%v nextSeq:%d", key, half.nextSeq)
	}
}
```

```
type StreamFactory interface {
	// New should return a new stream for the given TCP key.
	New(netFlow, tcpFlow gopacket.Flow, tcp *layers.TCP, ac AssemblerContext) Stream
}
```

```
// Stream is implemented by the caller to handle incoming reassembled
// TCP data.  Callers create a StreamFactory, then StreamPool uses
// it to create a new Stream for every TCP stream.
//
// assembly will, in order:
//    1) Create the stream via StreamFactory.New
//    2) Call ReassembledSG 0 or more times, passing in reassembled TCP data in order
//    3) Call ReassemblyComplete one time, after which the stream is dereferenced by assembly.
type Stream interface {
	// Tell whether the TCP packet should be accepted, start could be modified to force a start even if no SYN have been seen
	Accept(tcp *layers.TCP, ci gopacket.CaptureInfo, dir TCPFlowDirection, nextSeq Sequence, start *bool, ac AssemblerContext) bool

	// ReassembledSG is called zero or more times.
	// ScatterGather is reused after each Reassembled call,
	// so it's important to copy anything you need out of it,
	// especially bytes (or use KeepFrom())
	ReassembledSG(sg ScatterGather, ac AssemblerContext)

	// ReassemblyComplete is called when assembly decides there is
	// no more data for this Stream, either because a FIN or RST packet
	// was seen, or because the stream has timed out without any new
	// packet data (due to a call to FlushCloseOlderThan).
	// It should return true if the connection should be removed from the pool
	// It can return false if it want to see subsequent packets with Accept(), e.g. to
	// see FIN-ACK, for deeper state-machine analysis.
	ReassemblyComplete(ac AssemblerContext) bool
}
```

