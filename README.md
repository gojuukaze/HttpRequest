HttpRequest
=======
A simple `HTTP Request` package for golang. `GET` `POST` `DELETE` `PUT` `Upload`
fork from [github.com/kirinlabs/HttpRequest](github.com/kirinlabs/HttpRequest) 
change some method


### Installation
```bash
go get github.com/gojuukaze/HttpRequest

```

### Difference
```go
// gojuukaze/HttpRequest
m,err := res.JsonMap() //Format the json return map
log.Println(u)

// kirinlabs/HttpRequest
var m [string]interface{}
err := res.Json(&m)

//

// gojuukaze/HttpRequest
s,err := res.JsonString() //Format the json return string
log.Println(s)

// kirinlabs/HttpRequest
s,err := res.Export()

### How do we use HttpRequest?

#### Create request object use http.DefaultTransport
```go
req := HttpRequest.NewRequest()
req := HttpRequest.NewRequest().Debug(true).SetTimeout(5)
```

#### Set headers
```go
req.SetHeaders(map[string]string{
    "Content-Type": "application/x-www-form-urlencoded",
    "Connection": "keep-alive",
})

req.SetHeaders(map[string]string{
    "Source":"api",
})
```

#### Set cookies
```go
req.SetCookies(map[string]string{
    "name":"json",
    "token":"",
})

req.SetCookies(map[string]string{
    "age":"19",
})
```

#### Set basic auth
```go
req.SetBasicAuth("username","password")
```

#### Set timeout
```go
req.SetTimeout(5)  //default 30s
```

#### Transport
If you want to customize the Client object and reuse TCP connections, you need to define a global http.RoundTripper or & http.Transport, because the reuse of the http connection pool is based on Transport.
```go
var transport *http.Transport
func init() {   
    transport = &http.Transport{
        DialContext: (&net.Dialer{
            Timeout:   30 * time.Second,
            KeepAlive: 30 * time.Second,
            DualStack: true,
        }).DialContext,
        MaxIdleConns:          100,
    }
}

func demo(){
    // Use http.DefaultTransport
    res, err := HttpRequest.Get("http://127.0.0.1:8080")
    // Use custom Transport
    res, err := HttpRequest.Transport(transport).Get("http://127.0.0.1:8080")
}
```

#### Keep Alives，Only effective for custom Transport
```go
req.DisableKeepAlives(false)
res, err := HttpRequest.Transport(transport).DisableKeepAlives(false).Get("http://127.0.0.1:8080")
```

#### Ignore Https certificate validation，Only effective for custom Transport
```go
req.SetTLSClient(&tls.Config{InsecureSkipVerify: true})
res, err := HttpRequest.Transport(transport).SetTLSClient(&tls.Config{InsecureSkipVerify: true}).Get("http://127.0.0.1:8080")
```

#### Object-oriented operation mode
```go
req := HttpRequest.NewRequest().
	Debug(true).
	SetHeaders(map[string]string{
	    "Content-Type": "application/x-www-form-urlencoded",
	}).SetTimeout(5)
res,err := HttpRequest.NewRequest().Get("http://127.0.0.1")
```

### GET

#### Query parameter
```go
res, err := req.Get("http://127.0.0.1:8000")
res, err := req.Get("http://127.0.0.1:8000?id=10&title=HttpRequest")
res, err := req.Get("http://127.0.0.1:8000?id=10&title=HttpRequest",nil)
res, err := req.Get("http://127.0.0.1:8000?id=10&title=HttpRequest","address=beijing")

res, err := HttpRequest.Get("http://127.0.0.1:8000")
res, err := HttpRequest.Debug(true).SetHeaders(map[string]string{}).Get("http://127.0.0.1:8000")
```


#### Multi parameter
```go
res, err := req.Get("http://127.0.0.1:8000?id=10&title=HttpRequest",map[string]interface{}{
    "name":  "jason",
    "score": 100,
})

body, err := res.Body()
if err != nil {
    return
}

return string(body)
```


### POST

```go
res, err := req.Post("http://127.0.0.1:8000")
res, err := req.Post("http://127.0.0.1:8000", "title=github&type=1")
res, err := req.JSON().Post("http://127.0.0.1:8000", "{\"id\":10,\"title\":\"HttpRequest\"}")
res, err := req.Post("http://127.0.0.1:8000", map[string]interface{}{
    "id":    10,
    "title": "HttpRequest",
})
body, err := res.Body()
if err != nil {
    return
}
return string(body)

res, err := HttpRequest.Post("http://127.0.0.1:8000")
res, err := HttpRequest.JSON().Post("http://127.0.0.1:8000",map[string]interface{}{"title":"github"})
res, err := HttpRequest.Debug(true).SetHeaders(map[string]string{}).JSON().Post("http://127.0.0.1:8000","{\"title\":\"github\"}")
```

### Jar
```go
j, _ := cookiejar.New(nil)
j.SetCookies(&url.URL{
	Scheme: "http",
	Host:   "127.0.0.1:8000",
}, []*http.Cookie{
	&http.Cookie{Name: "identity-user", Value: "83df5154d0ed31d166f5c54ddc"},
	&http.Cookie{Name: "token_id", Value: "JSb99d0e7d809610186813583b4f802a37b99d"},
})
res, err := HttpRequest.Jar(j).Get("http://127.0.0.1:8000/city/list")
defer res.Close()
if err != nil {
	log.Fatalf("Request error：%v", err.Error())
}
```

### Proxy
```go
proxy, err := url.Parse("http://proxyip:proxyport")
if err != nil {
	log.Println(err)
}

res, err := HttpRequest.Proxy(http.ProxyURL(proxy)).Get("http://127.0.0.1:8000/ip")
defer res.Close()
if err != nil {
	log.Println("Request error：%v", err.Error())
}
body, err := res.Body()
if err != nil {
	log.Println("Get body error：%v", err.Error())
}
log.Println(string(body))
```

### Upload
Params: url, filename, fileinput

```go
res, err := req.Upload("http://127.0.0.1:8000/upload", "/root/demo.txt","uploadFile")
body, err := res.Body()
if err != nil {
    return
}
return string(body)
```


### Debug
#### Default false

```go
req.Debug(true)
```

#### Print in standard output：
```go
[HttpRequest]
-------------------------------------------------------------------
Request: GET http://127.0.0.1:8000?name=iceview&age=19&score=100
Headers: map[Content-Type:application/x-www-form-urlencoded]
Cookies: map[]
Timeout: 30s
ReqBody: map[age:19 score:100]
-------------------------------------------------------------------
```


## Json
Post JSON request

#### Set header
```go
 req.SetHeaders(map[string]string{"Content-Type": "application/json"})
```
Or
```go
req.JSON().Post("http://127.0.0.1:8000", map[string]interface{}{
    "id":    10,
    "title": "github",
})

req.JSON().Post("http://127.0.0.1:8000", "{\"title\":\"github\",\"id\":10}")
```

#### Post request
```go
res, err := req.Post("http://127.0.0.1:8000", map[string]interface{}{
    "id":    10,
    "title": "HttpRequest",
})
```

#### Print formatted JSON
```go
str, err := res.Export()
if err != nil {
   return
}
```

#### Unmarshal JSON
```go
var u User
err := res.Json(&u)
if err != nil {
   return err
}

var m map[string]interface{}
err := res.Json(&m)
if err != nil {
   return err
}
```

### Response

#### Response() *http.Response
```go
res, err := req.Post("http://127.0.0.1:8000/") //res is a http.Response object
```

#### StatusCode() int
```go
res.StatusCode()
```

#### Body() ([]byte, error)
```go
body, err := res.Body()
log.Println(string(body))
```

#### Close() error
```go
res.Close()
```

#### Time() string
```go
res.Time()  //ms
```

#### Print formatted JSON
```go
str, err := res.Export()
if err != nil {
   return
}
```

#### Unmarshal JSON
```go
var u User
err := res.Json(&u)
if err != nil {
   return err
}

var m map[string]interface{}
err := res.Json(&m)
if err != nil {
   return err
}
```

#### Url() string
```go
res.Url()  //return the requested url
```

#### Headers() http.Header
```go
res.Headers()  //return the response headers
```

#### Cookies() []*http.Cookie
```go
res.Cookies()  //return the response cookies
```

### Advanced
#### GET
```go
import "github.com/kirinlabs/HttpRequest"
   
res,err := HttpRequest.Get("http://127.0.0.1:8000/")
res,err := HttpRequest.Get("http://127.0.0.1:8000/","title=github")
res,err := HttpRequest.Get("http://127.0.0.1:8000/?title=github")
res,err := HttpRequest.Debug(true).JSON().Get("http://127.0.0.1:8000/")
```

#### POST
```go
import "github.com/kirinlabs/HttpRequest"
   
res,err := HttpRequest.Post("http://127.0.0.1:8000/")
res,err := HttpRequest.SetHeaders(map[string]string{
	"title":"github",
}).Post("http://127.0.0.1:8000/")
res,err := HttpRequest.Debug(true).JSON().Post("http://127.0.0.1:8000/")
```


### Example
```go
import "github.com/kirinlabs/HttpRequest"
   
res,err := HttpRequest.Get("http://127.0.0.1:8000/")
res,err := HttpRequest.Get("http://127.0.0.1:8000/","title=github")
res,err := HttpRequest.Get("http://127.0.0.1:8000/?title=github")
res,err := HttpRequest.Get("http://127.0.0.1:8000/",map[string]interface{}{
	"title":"github",
})
res,err := HttpRequest.Debug(true).JSON().SetHeaders(map[string]string{
	"source":"api",
}).SetCookies(map[string]string{
	"name":"httprequest",
}).Post("http://127.0.0.1:8000/")


//Or
req := HttpRequest.NewRequest()
req := req.Debug(true).SetHeaders()
res,err := req.Debug(true).JSON().SetHeaders(map[string]string{
    "source":"api",
}).SetCookies(map[string]string{
    "name":"httprequest",
}).Post("http://127.0.0.1:8000/")
```
