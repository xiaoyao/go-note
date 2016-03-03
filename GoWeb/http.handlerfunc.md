# http.HandlerFunc

## 1.type Handler
- ### type Handler interface

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```
Objects implementing the Handler interface can be registered to serve a particular path or subtree in the HTTP server.

ServeHTTP should write reply headers and data to the ResponseWriter and then return. Returning signals that the request is finished and that the HTTP server can move on to the next request on the connection.

If ServeHTTP panics, the server (the caller of ServeHTTP) assumes that the effect of the panic was isolated to the active request. It recovers the panic, logs a stack trace to the server error log, and hangs up the connection.

## 2.type HandlerFunc

- ###type HandlerFunc
```go
type HandlerFunc func(ResponseWriter, *Request)
```
The HandlerFunc type is an adapter to allow the use of ordinary functions as HTTP handlers. If f is a function with the appropriate signature, HandlerFunc(f) is a Handler object that calls f.

func (HandlerFunc) ServeHTTP

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request)
ServeHTTP calls f(w, r).
### 原码：
``` go
package http
~~~

// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```
- ListenAndServe starts an HTTP server with a given address and handler. The handler is usually nil, which means to use DefaultServeMux. Handle and HandleFunc add handlers to DefaultServeMux:

```go

http.Handle("/foo", fooHandler)

http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})

log.Fatal(http.ListenAndServe(":8080", nil))
```





- ### func Handle
```go
func Handle(pattern string, handler Handler)
```
Handle registers the handler for the given pattern in the DefaultServeMux. The documentation for ServeMux explains how patterns are matched.

- ### func HandleFunc
- 
```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```
HandleFunc registers the handler function for the given pattern in the DefaultServeMux. The documentation for ServeMux explains how patterns are matched.

- ###func ListenAndServe
```go
func ListenAndServe(addr string, handler Handler) error
```
ListenAndServe listens on the TCP network address addr and then calls Serve with handler to handle requests on incoming connections. Handler is typically nil, in which case the DefaultServeMux is used.

> A trivial example server is:

```go
package main

import (
	"io"
	"net/http"
	"log"
)

// hello world, the web server
func HelloServer(w http.ResponseWriter, req *http.Request) {
	io.WriteString(w, "hello, world!\n")
}

func main() {
	http.HandleFunc("/hello", HelloServer)
	err := http.ListenAndServe(":12345", nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
```