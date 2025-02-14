[![GoDoc](https://pkg.go.dev/badge/github.com/ConfusedPolarBear/serial.svg)](https://pkg.go.dev/github.com/ConfusedPolarBear/serial)

Serial
========
A Go package to allow you to read and write from a
serial port as a stream of bytes. Forked from
[tarm's serial library](https://github.com/tarm/serial).

Details
-------
It aims to have the same API on all platforms, including windows.  As
an added bonus, the windows package does not use cgo, so you can cross
compile for windows from another platform.

You can cross compile with
   GOOS=windows GOARCH=386 go install github.com/ConfusedPolarBear/serial

Currently there is very little in the way of configurability.  You can
set the baud rate.  Then you can Read(), Write(), or Close() the
connection.  By default Read() will block until at least one byte is
returned.  Write is the same.

Currently all ports are opened with 8 data bits, 1 stop bit, no
parity, no hardware flow control, and no software flow control.  This
works fine for many real devices and many faux serial devices
including usb-to-serial converters and bluetooth serial ports.

You may Read() and Write() simulantiously on the same connection (from
different goroutines).

Usage
-----
```go
package main

import (
        "log"

        "github.com/ConfusedPolarBear/serial"
)

func main() {
        c := &serial.Config{Name: "COM45", Baud: 115200}
        s, err := serial.OpenPort(c)
        if err != nil {
                log.Fatal(err)
        }
        
        n, err := s.Write([]byte("test"))
        if err != nil {
                log.Fatal(err)
        }
        
        buf := make([]byte, 128)
        n, err = s.Read(buf)
        if err != nil {
                log.Fatal(err)
        }
        log.Printf("%q", buf[:n])
}
```

NonBlocking Mode
----------------
By default the returned Port reads in blocking mode. If that's not
what you want, specify a positive ReadTimeout and Read() will timeout
and return 0 bytes if nothing is read. Note that this is the total
timeout for the read operation and not the interval timeout
between two bytes.

```go
	c := &serial.Config{Name: "COM45", Baud: 115200, ReadTimeout: time.Second * 5}
	
	// In this mode, you will want to suppress error for read
	// as 0 bytes return EOF error on Linux / POSIX
	n, _ = s.Read(buf)
```

Possible Future Work
-------------------- 
- better tests (loopback etc)
