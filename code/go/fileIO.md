# 文件读取

### ioutil 将文件全部放到内存中读写

```
func main() {
   file, err := os.Open("D:/gopath/src/golang_development_notes/example/log.txt")
   if err != nil {
      panic(err)
   }
   defer file.Close()
   content, err := ioutil.ReadAll(file)
   fmt.Println(string(content))
}
```

```
func main() {
   filepath := "D:/gopath/src/golang_development_notes/example/log.txt"
   content ,err :=ioutil.ReadFile(filepath)
   if err !=nil {
      panic(err)
   }
   fmt.Println(string(content))
}
```

### bufio 建立缓冲区，按size读入

```
func main() {
   filepath := "D:/gopath/src/golang_development_notes/example/log.txt"
   fi, err := os.Open(filepath)
   if err != nil {
      panic(err)
   }
   defer fi.Close()
   r := bufio.NewReader(fi)

   chunks := make([]byte, 0)
   buf := make([]byte, 1024) //一次读取多少个字节
   for {
      n, err := r.Read(buf)
      if err != nil && err != io.EOF {
         panic(err)
      }
      fmt.Println(string(buf[:n]))
      break
      if 0 == n {
         break
      }
      chunks = append(chunks, buf[:n]...)
   }
   fmt.Println(string(chunks))
}
```

```
func main() {

   file := "D:/gopath/src/golang_development_notes/example/log.txt"
   f, err := os.Open(file)
   if err != nil {
      panic(err)
   }
   defer f.Close()

   chunks := make([]byte, 0)
   buf := make([]byte, 1024)
   for {
      n, err := f.Read(buf)
      if err != nil && err != io.EOF {
         panic(err)
      }
      if 0 == n {
         break
      }
      chunks = append(chunks, buf[:n]...)
   }
   fmt.Println(string(chunks))
}
```
