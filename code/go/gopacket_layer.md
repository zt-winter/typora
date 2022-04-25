# 在使用gopacket时部分学习思考

### layer总体结构

Layer(interface) -> TCP(struct) -> BaseLayer(struct)
Layer(interface):定义三个方法 LayerType(),LayerContents(),LayerPayload()
TCP(struct):定义LayerType()
BaseLayer(struct):定义LayerContents(),LayerPayload()


### 接口的使用，动态类型推断

layers，也就是tcp/ip协议中各层，主要是链路层、网络层、传输层、应用层，在gopacket中具体指向流量数据某一层的协议。



在gopacket的说明文档中可以看到type Layer的定义

https://github.com/google/gopacket/blob/v1.1.19/base.go#L19

```go
type Layer interface {
	// LayerType is the gopacket type for this layer.
	LayerType() LayerType
	// LayerContents returns the set of bytes that make up this layer.
	LayerContents() []byte
	// LayerPayload returns the set of bytes contained within this layer, not
	// including the layer itself.
	LayerPayload() []byte
}
```

在实际使用时，可以看到下面的列子，**ethernetLayer.(*layers.Ethernet)**，需要对接口类型进行断言和转化。

```go
    ethernetLayer := packet.Layer(layers.LayerTypeEthernet)
    if ethernetLayer != nil {
        fmt.Println("Ethernet layer detected.")
        ethernetPacket, _ := ethernetLayer.(*layers.Ethernet)
        fmt.Println("Source MAC: ", ethernetPacket.SrcMAC)
        fmt.Println("Destination MAC: ", ethernetPacket.DstMAC)
        // Ethernet type is typically IPv4 but could be ARP or other
        fmt.Println("Ethernet type: ", ethernetPacket.EthernetType)
        fmt.Println()
    }
```



https://github.com/google/gopacket/blob/v1.1.19/layers/base.go 中可以看到BaseLayer的定义

```go
// BaseLayer is a convenience struct which implements the LayerData and
// LayerPayload functions of the Layer interface.
type BaseLayer struct {
	// Contents is the set of bytes that make up this layer.  IE: for an
	// Ethernet packet, this would be the set of bytes making up the
	// Ethernet frame.
	Contents []byte
	// Payload is the set of bytes contained by (but not part of) this
	// Layer.  Again, to take Ethernet as an example, this would be the
	// set of bytes encapsulated by the Ethernet protocol.
	Payload []byte
}
```

在BaseLayer的基础上，对不同层的不同协议进行

ipv4 layer的定义 https://github.com/google/gopacket/blob/v1.1.19/layers/ip4.go#L43 

```
type IPv4 struct {
	BaseLayer
	Version    uint8
	IHL        uint8
	TOS        uint8
	Length     uint16
	Id         uint16
	Flags      IPv4Flag
	FragOffset uint16
	TTL        uint8
	Protocol   IPProtocol
	   uint16
	SrcIP      net.IP
	DstIP      net.IP
	Options    []IPv4Option
	Padding    []byte
}
```

在Layer接口的实现中还缺少了LayerType()函数，具体实现可以参考下文

https://github.com/google/gopacket/blob/v1.1.19/layers/layertypes.go#L24 

```
LayerTypeIPv4 = gopacket.RegisterLayerType(20, gopacket.LayerTypeMetadata{Name: "IPv4", Decoder: gopacket.DecodeFunc(decodeIPv4)})
LayerTypeIPv6 = gopacket.RegisterLayerType(21, gopacket.LayerTypeMetadata{Name: "IPv6", Decoder: gopacket.DecodeFunc(decodeIPv6)})
```
https://github.com/google/gopacket/blob/a9779d139771f6a06fc983b18e0efd23ca30222f/layertype.go#L53

```
func RegisterLayerType(num int, meta LayerTypeMetadata) LayerType {
	if 0 <= num && num < maxLayerType {
		if ltMeta[num].inUse {
			panic("Layer type already exists")
		}
	} else {
		if ltMetaMap[LayerType(num)].inUse {
			panic("Layer type already exists")
		}
	}
	return OverrideLayerType(num, meta)
}
```

​	
