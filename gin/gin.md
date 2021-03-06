Index

- <a id="data_structure">Data Structure</a>
	- <a id="engine">Engine</a> 




----------

# Data Structure   [Return Parent](../README.md)
1.Engine (#engine)

```go
	// Engine is the framework's instance, it contains the muxer, middleware and configuration settings.
// Create an instance of Engine, by using New() or Default()
type Engine struct {
  RouterGroup

  // Enables automatic redirection if the current route can't be matched but a
 1 gin.go                                                                                                                                                                                                                          Buffers 
  // handler for the path with (without) the trailing slash exists.
  // For example if /foo/ is requested but a route only exists for /foo, the
  // client is redirected to /foo with http status code 301 for GET requests
  // and 307 for all other request methods.
  RedirectTrailingSlash bool

  // If enabled, the router tries to fix the current request path, if no
  // handle is registered for it.
  // First superfluous path elements like ../ or // are removed.
  // Afterwards the router does a case-insensitive lookup of the cleaned path.
  // If a handle can be found for this route, the router makes a redirection
  // to the corrected path with status code 301 for GET requests and 307 for
  // all other request methods.
  // For example /FOO and /..//Foo could be redirected to /foo.
  // RedirectTrailingSlash is independent of this option.
  RedirectFixedPath bool

  // If enabled, the router checks if another method is allowed for the
  // current route, if the current request can not be routed.
  // If this is the case, the request is answered with 'Method Not Allowed'
  // and HTTP status code 405.
  // If no other Method is allowed, the request is delegated to the NotFound
  // handler.
  HandleMethodNotAllowed bool

  // If enabled, client IP will be parsed from the request's headers that
  // match those stored at `(*gin.Engine).RemoteIPHeaders`. If no IP was
  // fetched, it falls back to the IP obtained from
  // `(*gin.Context).Request.RemoteAddr`.
  ForwardedByClientIP bool

  // List of headers used to obtain the client IP when
  // `(*gin.Engine).ForwardedByClientIP` is `true` and
  // `(*gin.Context).Request.RemoteAddr` is matched by at least one of the
  // network origins of `(*gin.Engine).TrustedProxies`.
  RemoteIPHeaders []string

  // List of network origins (IPv4 addresses, IPv4 CIDRs, IPv6 addresses or
  // IPv6 CIDRs) from which to trust request's headers that contain
  // alternative client IP when `(*gin.Engine).ForwardedByClientIP` is
  // `true`.
  TrustedProxies []string
    // #726 #755 If enabled, it will trust some headers starting with
  // 'X-AppEngine...' for better integration with that PaaS.
  
  AppEngine bool
  
  // If enabled, the url.RawPath will be used to find parameters.
  UseRawPath bool
  
  // If true, the path value will be unescaped.
  // If UseRawPath is false (by default), the UnescapePathValues effectively is true,
  // as url.Path gonna be used, which is already unescaped.
  UnescapePathValues bool 
  
  // Value of 'maxMemory' param that is given to http.Request's ParseMultipartForm
  // method call.
  MaxMultipartMemory int64
  
  // RemoveExtraSlash a parameter can be parsed from the URL even with extra slashes.
  // See the PR #1817 and issue #1644 
  RemoveExtraSlash bool
  
  delims           render.Delims
  secureJSONPrefix string
  HTMLRender       render.HTMLRender
  FuncMap          template.FuncMap
  allNoRoute       HandlersChain
  allNoMethod      HandlersChain
  noRoute          HandlersChain
  noMethod         HandlersChain
  pool             sync.Pool
  trees            methodTrees
  maxParams        uint16
  trustedCIDRs     []*net.IPNet
}
```

