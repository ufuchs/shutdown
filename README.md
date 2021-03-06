⏲️ shutdown
================
[![Build Status](https://travis-ci.org/vardius/shutdown.svg?branch=master)](https://travis-ci.org/vardius/shutdown)
[![Go Report Card](https://goreportcard.com/badge/github.com/vardius/shutdown)](https://goreportcard.com/report/github.com/vardius/shutdown)
[![codecov](https://codecov.io/gh/vardius/shutdown/branch/master/graph/badge.svg)](https://codecov.io/gh/vardius/shutdown)
[![](https://godoc.org/github.com/vardius/shutdown?status.svg)](https://pkg.go.dev/github.com/vardius/shutdown)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/vardius/shutdown/blob/master/LICENSE.md)

<img align="right" height="180px" src="https://github.com/vardius/gorouter/blob/master/website/src/static/img/logo.png?raw=true" alt="logo" />

shutdown - Simple go signals handler for performing graceful shutdown by executing callback function

📖 ABOUT
==================================================
Contributors:

* [Rafał Lorenz](http://rafallorenz.com)

Want to contribute ? Feel free to send pull requests!

Have problems, bugs, feature ideas?
We are using the github [issue tracker](https://github.com/vardius/shutdown/issues) to manage them.

## 📚 Documentation

For __examples__ **visit [godoc#pkg-examples](http://godoc.org/github.com/vardius/shutdown#pkg-examples)**

For **GoDoc** reference, **visit [pkg.go.dev](https://pkg.go.dev/github.com/vardius/shutdown)**

🚏 HOW TO USE
==================================================

For detailed breakdown of example [How to handle signals with Go to graceful shutdown HTTP server](https://rafallorenz.com/go/handle-signals-to-graceful-shutdown-http-server/)

## 🏫 Basic example
```go
package main

import (
	"context"
	"fmt"
	"log"
	"net"
	"net/http"
	"os"
	"syscall"
	"time"

    "github.com/vardius/shutdown"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
		fmt.Fprintf(w, "Hello!")
	})

	httpServer := &http.Server{
		Addr:    ":8080",
		Handler: mux,
		BaseContext: func(_ net.Listener) context.Context { return ctx },
	}

	stop := func() {
		gracefulCtx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
		defer cancel()

		if err := httpServer.Shutdown(gracefulCtx); err != nil {
			log.Printf("shutdown error: %v\n", err)
		} else {
			log.Printf("gracefully stopped\n")
		}
	}

	// Run server
	go func() {
		if err := httpServer.ListenAndServe(); err != http.ErrServerClosed {
			log.Printf("HTTP server ListenAndServe: %v", err)
			os.Exit(1)
		}
	}()

	shutdown.GracefulStop(stop) // will block until shutdown signal is received
}
```

📜 [License](LICENSE.md)
-------

This package is released under the MIT license. See the complete license in the package
