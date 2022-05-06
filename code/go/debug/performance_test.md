# 性能测试

### pprof

```
package main

import (
	"runtime/pprof"
	"os"
	"trafficAnalysis/config"
	"trafficAnalysis/extract"
)


func main() {
	pprof.StartCPUProfile(os.Stdout)
	defer pprof.StopCPUProfile()
	config := config.ReadConfig()
	extract.ExtractFeature(config)
}
```

```
go build
./trafficAnalysis -> cpu.pprof
go tool pprof -http 127.0.0.1:9999 cpu.pprof
```

![pprof火焰图](/home/zt/Pictures/pprof.png)
