---
title: "Golang Encrypt Io Copy"
date: 2019-10-30T19:12:51+08:00
draft: false
---

很多时候在两个机房传输数据的时候需要加密传输，加密传输的好处是防止中间人窃取信息。

记录一个简单的加密传输思路，当我们在两个机器上创建好tcp通道时，如果需要传输数据，需要用到io.Copy，这里我自定义一个Copy，实现加密。简单靠谱

```golang
func Copy(dst io.Writer, src io.Reader) (written int64, err error) {
	buf := make([]byte, 1024)
	request := make([]byte, 1024)
	for {
		nr, er := src.Read(buf)
		if nr > 0 {
			request = key(buf[0:nr])
			nw, ew := dst.Write(request)
			if nw > 0 {
				written += int64(nw)
			}
			if ew != nil {
				err = ew
				break
			}
			if nr != nw {
				err = io.ErrShortWrite
				break
			}
		}
		if er != nil {
			if er != io.EOF {
				err = er
			}
			break
		}
	}
	return written, err
}
```

里面的key方法对当前buffer进行加密后写入，如果src是密文，则功能转变为解密

