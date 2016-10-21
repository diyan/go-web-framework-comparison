# Golang Web Framework Comparsion

TODO put explanation and motivation here.

### Contents
- [HTTP handler. Signature](#http-handler-signature)
    - [tl;dr](#http-handler-signature-tldr)
    - stdlib net/http, gorilla/mux, urfave/negroni
    - gin-gonic/gin
    - go-macaron/macaron
    - go-martini/martini
    - gocraft/web
    - goji/goji
    - hoisie/web
    - julienschmidt/httprouter
    - labstack/echo
    - pressly/chi
    - [zenazn/goji](#zenazn--goji)
- [HTTP middleware. Signature and sample code](#http-middleware-signature-and-sample-code)
    - [tl;dr](#http-middeware-signature-tldr)
    - stdlib net/http, gorilla/mux, pressly/chi
    - gin-gonic/gin
    - gocraft/web
    - goji/goji 
    - labstack/echo
    - urfave/negroni
- [HTTP handler. Write Go struct as JSON response](#http-handler-write-go-struct-as-json-response)
    - tl;dr
    - common part
    - stdlib net/http, gorilla/mux, negroni, zenazn/goji, goji/goji, julienschmidt/httprouter. V1 - use only stdlib
    - stdlib net/http, gorilla/mux, negroni, zenazn/goji, goji/goji, julienschmidt/httprouter. V2 - use unrolled/render that mentioned in negroni's README
    - gin-gonic/gin
    - go-macaron/macaron
    - go-martini/martini
    - gocraft/web
    - hoisie/web
    - labstack/echo
    - pressly/chi    
- [HTTP handler. Bind JSON payload into Go struct](#http-handler-bind-json-payload-into-go-struct)
    - tl;dr
    - common part
    - stdlib net/http, gorilla/mux, negroni, zenazn/goji, goji/goji, julienschmidt/httprouter, go-macaron/macaron, hoisie/web. V1 - use only stdlib
    - stdlib net/http, gorilla/mux, negroni, zenazn/goji, goji/goji, julienschmidt/httprouter, go-macaron/macaron, hoisie/web. V2 - use mholt/binding
    - gin-gonic/gin
    - go-martini/martini
    - gocraft/web
    - labstack/echo
    - pressly/chi

## HTTP handler. Signature

### tl;dr
TODO put explanation and summary here. Raw notes are following:

CON! revel, beego are not idiomatic Go because they forces you to embed handler to the Framework's struct

CON martini, hoisie/web, macaron handlers are not strongly typed due to reflective dependency injection (which leads to poor performance)

CON zenazn/goji handler and middleware is not strongly typed to emulate function overload

CON gocraft/web handler is not strongly typed because it's a method with pointer receiver that could be any user-defined struct

CON negroni, stdlib net/http does not dispatch request by HTTP verb (more boilerplate code)

CON hoisie/web, zenazn/goji offers to use singletone istance of the server struct which is quite bad practice

Q goji/goji has quite unusual API to dispatch requests by HTTP verb but it's still more verbose than in echo, gin, julienschmidt/httprouter

PRO echo, gin, julienschmidt/httprouter, zenazn/goji, goji/goji, pressly/chi handers are stronly typed

PRO echo, gin, julienschmidt/httprouter, zenazn/goji, pressly/chi do dispatch requests by HTTP verb

PRO echo, gin, zenazn/goji, pressly/chi support HTTP middleware
Q julienschmidt/httprouter does not support HTTP middleware, gorilla/handlers are recommended instead

Q goji/goji keeps handler interface standard but it's quite verbose to type. Consider

Q echo handers returns error value which could be handled in next middlewares in the chain. Consider


FYI labstack/echo has own router, supports most handler / middleware APIs

FYI gin uses julienschmidt/httprouter, per-request context map

FYI negroni recommends gorilla/mux router; Golang 1.7 context can be used (or gorilla/context for Golang < 1.7)

CON gorilla/context uses global context map which may lead to lock contention

PRO Golang 1.7 context uses per-request context map, Request.WithContext does a shallow copy of *Request

### stdlib net/http
stdlib net/http or negroni + stdlib net/http or negroni + gorilla/mux, https://golang.org/pkg/net/http/#ServeMux.HandleFunc
```golang
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))

mux.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) { ...
```

### gin-gonic/gin
https://godoc.org/github.com/gin-gonic/gin#RouterGroup.GET
```golang
func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes
type HandlerFunc func(*gin.Context)

g.GET("/", func(c *gin.Context) { ...
```

### go-macaron/macaron
https://godoc.org/github.com/go-macaron/macaron#Router.Get
```golang
func (r *Router) Get(pattern string, h ...Handler) (leaf *Route)
type Handler interface{}

m.Get("/", func(w http.ResponseWriter, req *http.Request, log *log.Logger) { ...
m.Get("/", func(ctx *macaron.Context) (int, []byte) { ...
```

### go-martini/martini
https://godoc.org/github.com/go-martini/martini#Router
```golang
func (r *Router) Get(string, ...Handler) Route
type Handler interface{}

m.Get("/", func(w http.ResponseWriter, r *http.Request, u *SomeUserService) { ...
m.Get("/", func() (int, string) { ...
```

### gocraft/web
https://godoc.org/github.com/gocraft/web#Router.Get
```golang
func (r *Router) Get(path string, fn interface{}) *Router

w.Get("/", func (c *SomeUserContext) SayHello(w web.ResponseWriter, r *web.Request) { ...
```

### goji/goji
https://godoc.org/github.com/goji/goji#Mux.HandleFunc
```golang
func (m *Mux) HandleFunc(p Pattern, h func(http.ResponseWriter, *http.Request))
func (m *Mux) HandleFuncC(p Pattern, h func(context.Context, http.ResponseWriter, *http.Request))
type Pattern interface {
    Match(context.Context, *http.Request) context.Context
}

g.HandleFunc(pat.Get("/"), func (w http.ResponseWriter, r *http.Request) { ...
g.HandleFuncC(pat.Get("/"), func (ctx context.Context, w http.ResponseWriter, r *http.Request) { ...
```

### hoisie/web
https://godoc.org/github.com/hoisie/web#Get
```golang
func Get(route string, handler interface{})

w.Get("/", func(ctx *web.Context, val string) { ...
w.Get("/", func(val string) string { ...
```

### julienschmidt/httprouter
https://godoc.org/github.com/julienschmidt/httprouter#Router.GET
```golang
func (r *Router) GET(path string, handle Handle)
type Handle func(http.ResponseWriter, *http.Request, Params)

r.GET("/", func Index(w http.ResponseWriter, r *http.Request, ps httprouter.Params) { ...
```

### labstack/echo
https://godoc.org/github.com/labstack/echo#Echo.Get
```golang
func (e *Echo) Get(path string, h HandlerFunc, m ...MiddlewareFunc)
type HandlerFunc func(echo.Context) error
type MiddlewareFunc func(HandlerFunc) HandlerFunc

e.Get("/", func(c echo.Context) error { ...
```

### pressly/chi
https://godoc.org/github.com/pressly/chi#Mux.Get
```golang
func (mx *Mux) Get(pattern string, handlerFn http.HandlerFunc)

c.Get("/", func(w http.ResponseWriter, r *http.Request) { ...
```

### zenazn/goji
https://godoc.org/github.com/zenazn/goji#Get
```golang
func Get(pattern web.PatternType, handler web.HandlerType)
type PatternType interface{}
type HandlerType interface{}

g.Get("/", func(w http.ResponseWriter, req *http.Request) { ...
```

## HTTP middleware. Signature and sample code

### tl;dr
TODO add godoc urls, summary. Raw notes are following:

CON revel, beego are not considered, they forces you to embed handler to the Framework's struct

CON martini, hoisie/web, macaron are not considered, their handers are not strongly typed due to reflective dependency injection

CON zenazn/goji handler and middleware are not strongly typed to emulate function overload

CON gin has "func (c *Context) Next()" function that visible to all handlers but must be called only inside middleware

PRO echo, goji/goji, pressly/chi, negroni has strongly typed middleware with reasonable signatures

Q negroni uses quite unusual signature for middleware. Why? I have only one explanation at this point. Author decide to avoid usage of higher-order function, so it would be easier to grasp for not-experienced developers

TODO gocraft/web PROs and CONs that related to the middleware API

### stdlib net/http, gorilla/mux, julienschmidt/httprouter, pressly/chi
```golang
func(http.Handler) http.Handler

func Middleware(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
	// do some stuff before
	next.ServeHTTP(w, r)
	// do some stuff after
  })
}
```

### gin-gonic/gin
```golang
// gin uses same signature for both handler and middleware
type HandlerFunc func(*Context)

func Middleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // do some stuff before
        c.Next()
        // do some stuff after
    }
}
```

### gocraft/web
```golang
// TODO

func (c *SomeUserContext) SetHelloCount(w web.ResponseWriter, r *web.Request, next web.NextMiddlewareFunc) {
    // do some stuff before
    next(w, r)
    // do some stuff after
}
```

### goji/goji 
```golang
func(http.Handler) http.Handler // standard middleware
func(goji.Handler) goji.Handler  // context-aware middleware

func Middleware(inner goji.Handler) goji.Handler {
	return goji.HandlerFunc(func(ctx context.Context, w http.ResponseWriter, r *http.Request) {
		// do some stuff before
		inner.ServeHTTPC(ctx, w, r)
		/ do some stuff after
	})
}
```

### labstack/echo
```golang
type MiddlewareFunc func(HandlerFunc) HandlerFunc
type HandlerFunc func(Context) error

func Middleware(next echo.HandlerFunc) echo.HandlerFunc {
	return func(c echo.Context) error {
		// do some stuff before
		err := next(c)
		// do some stuff after
		return err
	}
}
```

### urfave/negroni
```golang
type HandlerFunc func(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc)

func Middleware(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
  // do some stuff before
  next(rw, r)
  // do some stuff after
}
```

## HTTP handler. Write Go struct as JSON response
TODO

## HTTP handler. Bind JSON payload into Go struct
TODO
