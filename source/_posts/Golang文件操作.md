---
title: Golang文件操作
date: 2020-02-28 15:57:29
desc: 
tags: ['Golang']
---

## 判断文件是否纯在

```go
package main

import (
  "fmt"
  "os"
)

func main() {
  stat, err := os.Stat("~/Demo.txt")
  if err != nil {
    fmt.Println("err is", err)
    if os.IsNotExist(err) {
      fmt.Println("文件不纯在")
    }
  } else {
    fmt.Println("文件纯在!")
    fmt.Println(stat)
  }
}
```

## 文件间拷贝
```go
package main

import (
  "fmt"
  "io"
  "os"
)

func main() {
  srcFile, _ := os.OpenFile("~/in.txt")
  dstFile, _ := os.OpenFile("~/out.txt")
  written, err := io.Copy(dstFile, srcFile)
  if err == nil {
    fmt.Println("拷贝成功, 字节数=", written)
  } else {
    fmt.Println("拷贝失败，err=",err)
  }
}
```

## 缓冲式文件拷贝
```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

func main() {
	//打开源文件
	srcfile, _ := os.OpenFile("~/in.txt", os.O_RDONLY, 0666)
	//	打开目标文件
	dstfile, _ := os.OpenFile("~/out.txt", os.O_CREATE|os.O_WRONLY, 0666)
	defer func() {
		srcfile.Close()
		dstfile.Close()
		fmt.Println("文件已关闭")
	}()

	reader := bufio.NewReader(srcfile)
	writer := bufio.NewWriter(dstfile)
	buffer := make([]byte, 1024)
	for{
		_ , err := reader.Read(buffer)
		if err != nil{
			if err == io.EOF{
				fmt.Println("源文件读取完毕")
				break
			}else {
				fmt.Println("error")
				return
			}
		}else {
			_, err1 := writer.Write(buffer)
			if err1 != nil{
				fmt.Println(err1)
				return
			}
		}

	}
}
```