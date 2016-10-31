# Golang Web Framework Comparsion

This suite aims to compare the public API of various Go web frameworks and routers.

NOTE While code blocks are self-explained the list of PROs an CONs are highly opinionated and targeted Go 1.7+. Even if some frameworks has a more :thumbsdown: than another they still are awesome and may work better for your use cases.

## Contents
- [Reviewed libraries and frameworks](#reviewed-libraries-and-frameworks)
- [HTTP handler. Signature](#http-handler-signature)
- [HTTP middleware. Signature and sample code](#http-middleware-signature-and-sample-code)
- [HTTP handler. Write Go struct as JSON response](#http-handler-write-go-struct-as-json-response)
- [HTTP handler. Bind JSON payload into Go struct](#http-handler-bind-json-payload-into-go-struct)

## Reviewed libraries and frameworks

- stdlib net/http
- gin-gonic/gin
- go-macaron/macaron
- go-martini/martini
- go-ozzo/ozzo-routing
- gocraft/web
- goji/goji
- gorilla/mux
- hoisie/web
- julienschmidt/httprouter (router)
- labstack/echo
- mholt/binding (request binding)
- pressly/chi
- TODO astaxie/beego
- TODO revel/revel
- unrolled/render (response rendering)
- urfave/negroni
- zenazn/goji

## HTTP handler. Signature

- :thumbsdown: :exclamation: revel, beego are not idiomatic Go because they forces you to embed handler to the Framework's struct
- :thumbsdown: martini, hoisie/web, macaron handlers are not strongly typed due to reflective dependency injection (which leads to poor performance)
- :thumbsdown: zenazn/goji handler and middleware is not strongly typed to emulate function overload
- :thumbsdown: gocraft/web handler is not strongly typed because it's a method with pointer receiver that could be any user-defined struct
- :thumbsdown: negroni, stdlib net/http does not dispatch request by HTTP verb (more boilerplate code)
- :thumbsdown: hoisie/web, zenazn/goji offers to use singletone istance of the server struct which is quite bad practice
- :question: goji/goji has quite unusual API to dispatch requests by HTTP verb but it's still more verbose than in echo, gin, julienschmidt/httprouter
- :thumbsup: echo, gin, julienschmidt/httprouter, zenazn/goji, goji/goji, ozzo-routing, pressly/chi handers are stronly typed
- :thumbsup: echo, gin, julienschmidt/httprouter, zenazn/goji, ozzo-routing, pressly/chi do dispatch requests by HTTP verb
- :thumbsup: echo, gin, zenazn/goji, ozzo-routing, pressly/chi support HTTP middleware
- :thumbsup: echo, ozzo-routing handers returns error value which could be handled in next middlewares in the chain
- :question: julienschmidt/httprouter does not support HTTP middleware, gorilla/handlers are recommended instead
- :question: goji/goji keeps handler interface standard but it's quite verbose to type
- FYI labstack/echo has own router, supports most handler / middleware APIs
- FYI gin uses julienschmidt/httprouter, per-request context map
- FYI negroni recommends gorilla/mux router; Golang 1.7 context can be used (or gorilla/context for Golang < 1.7)
- :thumbsdown: gorilla/context uses global context map which may lead to lock contention
- :thumbsup: Golang 1.7 context uses per-request context map, Request.WithContext does a shallow copy of *Request

### stdlib net/http
stdlib net/http or negroni + stdlib net/http or negroni + gorilla/mux, https://golang.org/pkg/net/http/#ServeMux.HandleFunc
```go
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))

mux.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) { ...
```

### gin-gonic/gin
https://godoc.org/github.com/gin-gonic/gin#RouterGroup.GET
```go
func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes
type HandlerFunc func(*gin.Context)

g.GET("/", func(c *gin.Context) { ...
```

### go-macaron/macaron
https://godoc.org/github.com/go-macaron/macaron#Router.Get
```go
func (r *Router) Get(pattern string, h ...Handler) (leaf *Route)
type Handler interface{}

m.Get("/", func(w http.ResponseWriter, req *http.Request, log *log.Logger) { ...
m.Get("/", func(ctx *macaron.Context) (int, []byte) { ...
```

### go-martini/martini
https://godoc.org/github.com/go-martini/martini#Router
```go
func (r *Router) Get(string, ...Handler) Route
type Handler interface{}

m.Get("/", func(w http.ResponseWriter, r *http.Request, u *SomeUserService) { ...
m.Get("/", func() (int, string) { ...
```

### gocraft/web
https://godoc.org/github.com/gocraft/web#Router.Get
```go
func (r *Router) Get(path string, fn interface{}) *Router

w.Get("/", func (c *SomeUserContext) SayHello(w web.ResponseWriter, r *web.Request) { ...
```

### goji/goji
https://godoc.org/github.com/goji/goji#Mux.HandleFunc
```go
func (m *Mux) HandleFunc(p Pattern, h func(http.ResponseWriter, *http.Request))
func (m *Mux) HandleFuncC(p Pattern, h func(context.Context, http.ResponseWriter, *http.Request))
type Pattern interface {
    Match(context.Context, *http.Request) context.Context
}

g.HandleFunc(pat.Get("/"), func (w http.ResponseWriter, r *http.Request) { ...
g.HandleFuncC(pat.Get("/"), func (ctx context.Context, w http.ResponseWriter, r *http.Request) { ...
```

### go-ozzo/ozzo-routing
https://godoc.org/github.com/go-ozzo/ozzo-routing#RouteGroup.Get
```go
func (r *RouteGroup) Get(path string, handlers ...Handler) *Route
type Handler func(*routing.Context) error

r.Get("/", func(c *routing.Context) error { ...
```

### hoisie/web
https://godoc.org/github.com/hoisie/web#Get
```go
func Get(route string, handler interface{})

w.Get("/", func(ctx *web.Context, val string) { ...
w.Get("/", func(val string) string { ...
```

### julienschmidt/httprouter
https://godoc.org/github.com/julienschmidt/httprouter#Router.GET
```go
func (r *Router) GET(path string, handle Handle)
type Handle func(http.ResponseWriter, *http.Request, Params)

r.GET("/", func Index(w http.ResponseWriter, r *http.Request, ps httprouter.Params) { ...
```

### labstack/echo
https://godoc.org/github.com/labstack/echo#Echo.Get
```go
func (e *Echo) Get(path string, h HandlerFunc, m ...MiddlewareFunc)
type HandlerFunc func(echo.Context) error
type MiddlewareFunc func(HandlerFunc) HandlerFunc

e.Get("/", func(c echo.Context) error { ...
```

### pressly/chi
https://godoc.org/github.com/pressly/chi#Mux.Get
```go
func (mx *Mux) Get(pattern string, handlerFn http.HandlerFunc)

c.Get("/", func(w http.ResponseWriter, r *http.Request) { ...
```

### zenazn/goji
https://godoc.org/github.com/zenazn/goji#Get
```go
func Get(pattern web.PatternType, handler web.HandlerType)
type PatternType interface{}
type HandlerType interface{}

g.Get("/", func(w http.ResponseWriter, req *http.Request) { ...
```

## HTTP middleware. Signature and sample code

TODO add godoc urls

- :thumbsdown: revel, beego are not considered, they forces you to embed handler to the Framework's struct
- :thumbsdown: martini, hoisie/web, macaron are not considered, their handers are not strongly typed due to reflective dependency injection
- :thumbsdown: zenazn/goji handler and middleware are not strongly typed to emulate function overload
- :thumbsdown: gin has "func (c *Context) Next()" function that visible to all handlers but must be called only inside middleware
- :thumbsup: echo, goji/goji, ozzo-routing, pressly/chi, negroni has strongly typed middleware with reasonable signatures
- :question: negroni uses quite unusual signature for middleware. Why? I have only one explanation at this point. Author decide to avoid usage of higher-order function, so it would be easier to grasp for not-experienced developers
- TODO gocraft/web PROs and CONs that related to the middleware API

### stdlib net/http, gorilla/mux, julienschmidt/httprouter, pressly/chi
```go
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
```go
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
```go
// TODO

func (c *SomeUserContext) SetHelloCount(w web.ResponseWriter, r *web.Request, next web.NextMiddlewareFunc) {
    // do some stuff before
    next(w, r)
    // do some stuff after
}
```

### goji/goji 
```go
func(http.Handler) http.Handler // standard middleware
func(goji.Handler) goji.Handler  // context-aware middleware

func Middleware(inner goji.Handler) goji.Handler {
	return goji.HandlerFunc(func(ctx context.Context, w http.ResponseWriter, r *http.Request) {
		// do some stuff before
		inner.ServeHTTPC(ctx, w, r)
		// do some stuff after
	})
}
```

### go-ozzo/ozzo-routing
```go
// middlewares and handers share the same type "Handler" in ozzo-routing

func Middleware(c *routing.Context) error {
	// do some stuff before
	err := c.Next()
	// do some stuff after
	return err
}
```

### labstack/echo
```go
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
```go
type HandlerFunc func(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc)

func Middleware(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
  // do some stuff before
  next(rw, r)
  // do some stuff after
}
```

## HTTP handler. Write Go struct as JSON response

### Common part
```go
type Greeting struct {
    Hello string `json:"hello"`
}
```

### stdlib net/http, gorilla/mux, negroni, zenazn/goji, goji/goji, julienschmidt/httprouter. V1 - use only stdlib
```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json; charset=UTF-8")
    w.WriteHeader(http.StatusOK)    
    if err := json.NewEncoder(w).Encode(Greeting{Hello: "world"}); err != nil {
        panic(err)
    }
})
```

### stdlib net/http, gorilla/mux, negroni, zenazn/goji, goji/goji, julienschmidt/httprouter. V2 - use unrolled/render that mentioned in negroni's README
```go
r := render.New()
mux.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
    if err := r.JSON(w, http.StatusOK, Greeting{Hello: "world"}); err != nil {
        panic(err)
    }
})
```

### gin-gonic/gin
```go
r.GET("/", func(c *gin.Context) {
    // method will panic if serialization fails
    c.JSON(http.StatusOK, Greeting{Hello: "world"})
})
```

### go-macaron/macaron. Use either macaron.Context or macaron.render.Render
```go
// render.Render or macaron.Context will be passed to the handler by using dependency injection
m.Get("/", func(r render.Render) {
    // method will render HTTP 500 response if serialization fails
    r.JSON(http.StatusOK, Greeting{Hello: "world"})
})

// V2
m.Get("/", func(ctx *macaron.Context) {
    ctx.JSON(http.StatusOK, Greeting{Hello: "world"})
}
```

### go-martini/martini + martini-contrib/render that menioned in README
```go
// render.Render will be passed to the handler by using dependency injection
m.Get("/", func(r render.Render) {
    // method will render HTTP 500 response if serialization fails
    r.JSON(http.StatusOK, Greeting{Hello: "world"})
})
```

### go-ozzo/ozzo-routing
```go
// Context.Write will write the data in the format determined by the content negotiation middleware (included)
r.Get("/", func(c *routing.Context) error {
    // if Write returns an error, the framework will render HTTP 500 response by default
    return c.Write(Greeting{Hello: "world"})
})
```

### gocraft/web + corneldamian/json-binding than mentioned in README
TODO Review implementation in https://github.com/corneldamian/json-binding/blob/master/response.go#L47

### hoisie/web
```go
// web.Context will be passed to the handler by using dependency injection
w.Get("/", func(ctx *web.Context) string {
    ctx.ContentType("json")
    data, err := json.Marshal(Greeting{Hello: "world"})
    if err != nil {
        panic(err)
    }
    return string(data)
})

// V2
w.Get("/", func(ctx *Context) []byte {
    ctx.ContentType("json")
    data, err := json.Marshal(Greeting{Hello: "world"})
    if err != nil {
        panic(err)
    }
    return data
})
```

### labstack/echo
```go
e.Get("/", func(c echo.Context) error {
    //  method will return error if serialization fails, then handler return that error to the middleware/framework
    return c.JSON(http.StatusOK, Greeting{Hello: "world"})
})
```

### pressly/chi
```go
c.Get("/", func(w http.ResponseWriter, r *http.Request) {
    // method will render HTTP 500 response if serialization fails
    render.JSON(w, r, Greeting{Hello: "world"})
})
```

## HTTP handler. Bind JSON payload into Go struct

### Common part
```go
type Greeting struct {
	Hello string `json:"hello"`
}
```

### stdlib net/http, gorilla/mux, negroni, zenazn/goji, goji/goji, julienschmidt/httprouter, go-macaron/macaron, hoisie/web. V1 - use only stdlib
```go
mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
	g := new(Greeting)
	defer r.Body.Close()
	if err := json.NewDecoder(r.Body).Decode(g); err != nil {
		panic(err)
	}
	// TODO put here validation code
	w.WriteHeader(http.StatusCreated)
}
```

### stdlib net/http, gorilla/mux, negroni, zenazn/goji, goji/goji, julienschmidt/httprouter, go-macaron/macaron, hoisie/web. V2 - use mholt/binding
```go
// mholt/binding is the reflectioness data binding library developed by co-author of the martini-contrib/binding

// for JSON you can return an empty field map
func (g *Greeting) FieldMap() binding.FieldMap {
	return binding.FieldMap{}
}

mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
	g := new(Greeting)
	errs := binding.Bind(r, g)
	// If parsing failed Handle will render HTTP 400 response with error details and return true
	if errs.Handle(w) {
		return
	}
	w.WriteHeader(http.StatusCreated)
}
```

### gin-gonic/gin
```go
r.POST("/", func(c *gin.Context) {
	g := new(Greeting)
	if c.BindJSON(g); err != nil {
		panic(err)
	}
	c.Abort(http.StatusCreated)
})
```

### go-martini/martini + martini-contrib/binding that menioned in README
```go
// Greeting struct may also contain binding tags
type Greeting struct {
	Hello string `json:"hello" binding:"required"`
}
// TODO check is binding.Bind(..) renders HTTP 400 response if validation fails
m.Post("/", binding.Bind(Greeting{}), func(g Greeting) (int, string) {
	return http.StatusNoContent, ""
})
```

### go-ozzo/ozzo-routing
```go
r.Post("/", func(c *routing.Context) error {
	var g Greeting
	// Context.Read will read the data in the format specified by the "Content-Type" header
	if err := c.Read(&g); err != nil {
		return err
	}
	c.Response.WriteHeader(http.StatusCreated)
	return nil
})
```

### gocraft/web + corneldamian/json-binding than mentioned in README
TODO Review implementation in https://github.com/corneldamian/json-binding/blob/master/request.go#L56

### labstack/echo
```go
e.POST("/", func(c echo.Context) error {
	g := new(Greeting)
	if err := c.Bind(g); err != nil {
		return err
	}
	return c.NoContent(http.StatusCreated)
})
```

### pressly/chi
```go
c.Post("/", func(w http.ResponseWriter, r *http.Request) {
	g := new(Greeting)
	if err := render.Bind(r.Body, g); err != nil {
		panic(err)
	}
	render.NoContent(w, r)
})
```
