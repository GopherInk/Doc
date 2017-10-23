我 10 个月前开始成为一名 Gopher，没有回头。像许多其他 gopher 一样，我很快发现简单的语言特性对于快速构建快速、可扩展的软件非常有用。当我刚开始学习 Go 时，我正在玩不同的多路复用器（multiplexer），它可以作为 API 服务器使用。如果您像我一样有 Rails 背景，你可能也会在构建 Web 框架提供的所有功能方面遇到困难。回到多路复用器，我发现了 3 个是非常有用的好东西，即 Gorilla mux、httprouter 和 bone（按性能从低到高排列）。即使 bone 有最佳性能和更简单的 handler 签名，但对于我来说，它仍然不够成熟，无法用于生产环境。因此，我最终使用了 httprouter。在本教程中，我将使用 httprouter 构建一个简单的 REST API 服务器。

如果你想偷懒，只想获取源码，你可以在这里[4]直接检出我的 github 仓库。

让我们开始吧。首先创建一个基本端点：
```
package main

import (
    "fmt"
    "log"
    "net/http"

    "github.com/julienschmidt/httprouter"
)

func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
    fmt.Fprint(w, "Welcome!\n")
}

func main() {
    router := httprouter.New()
    router.GET("/", Index)

    log.Fatal(http.ListenAndServe(":8080", router))
}
```
在上面的代码段中，Index 是一个 handler 函数，需要传入三个参数。 之后，该 handler 将在 main 函数中被注册到 GET / 路径。 现在编译并运行您的程序，转到 http:// localhost:8080，来查看您的 API 服务器。点击这里[1]获取当前代码。

现在我们可以让 API 变得复杂一点。我们现在有一个名为 Book 的实体，可以把 ISDN 字段作为唯一标识。让我们创建更多的动作，即分表代表着 Index 和 Show 动作的 GET /books 和 GET /books/:isdn。 我们的 main.go 文件此时如下：
```
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"

    "github.com/julienschmidt/httprouter"
)

func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
    fmt.Fprint(w, "Welcome!\n")
}

type Book struct {
    // The main identifier for the Book. This will be unique.
    ISDN   string `json:"isdn"`
    Title  string `json:"title"`
    Author string `json:"author"`
    Pages  int    `json:"pages"`
}

type JsonResponse struct {
    // Reserved field to add some meta information to the API response
    Meta interface{} `json:"meta"`
    Data interface{} `json:"data"`
}

type JsonErrorResponse struct {
    Error *ApiError `json:"error"`
}

type ApiError struct {
    Status int16  `json:"status"`
    Title  string `json:"title"`
}

// A map to store the books with the ISDN as the key
// This acts as the storage in lieu of an actual database
var bookstore = make(map[string]*Book)

// Handler for the books index action
// GET /books
func BookIndex(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
    books := []*Book{}
    for _, book := range bookstore {
        books = append(books, book)
    }
    response := &JsonResponse{Data: &books}
    w.Header().Set("Content-Type", "application/json; charset=UTF-8")
    w.WriteHeader(http.StatusOK)
    if err := json.NewEncoder(w).Encode(response); err != nil {
        panic(err)
    }
}

// Handler for the books Show action
// GET /books/:isdn
func BookShow(w http.ResponseWriter, r *http.Request, params httprouter.Params) {
    isdn := params.ByName("isdn")
    book, ok := bookstore[isdn]
    w.Header().Set("Content-Type", "application/json; charset=UTF-8")
    if !ok {
        // No book with the isdn in the url has been found
        w.WriteHeader(http.StatusNotFound)
        response := JsonErrorResponse{Error: &ApiError{Status: 404, Title: "Record Not Found"}}
        if err := json.NewEncoder(w).Encode(response); err != nil {
            panic(err)
        }
    }
    response := JsonResponse{Data: book}
    if err := json.NewEncoder(w).Encode(response); err != nil {
        panic(err)
    }
}

func main() {
    router := httprouter.New()
    router.GET("/", Index)
    router.GET("/books", BookIndex)
    router.GET("/books/:isdn", BookShow)

    // Create a couple of sample Book entries
    bookstore["123"] = &Book{
        ISDN:   "123",
        Title:  "Silence of the Lambs",
        Author: "Thomas Harris",
        Pages:  367,
    }

    bookstore["124"] = &Book{
        ISDN:   "124",
        Title:  "To Kill a Mocking Bird",
        Author: "Harper Lee",
        Pages:  320,
    }

    log.Fatal(http.ListenAndServe(":8080", router))
}
```
如果您现在尝试请求 GET https:// localhost:8080/books，您将得到以下响应：
```
{
    "meta": null,
    "data": [
        {
            "isdn": "123",
            "title": "Silence of the Lambs",
            "author": "Thomas Harris",
            "pages": 367
        },
        {
            "isdn": "124",
            "title": "To Kill a Mocking Bird",
            "author": "Harper Lee",
            "pages": 320
        }
    ]
}
```
我们在 main 函数中硬编码了这两个 book 实体。点击这里[2]获取当前阶段的代码。

让我们来重构一下代码。 到目前为止，我们所有的代码都放置在同一个文件中：main.go。我们可以把它们移到各个单独的文件中。此时我们有一个目录：
```
.
├── handlers.go
├── main.go
├── models.go
└── responses.go
```
我们把所有与 JSON 响应相关的结构体移动到 responses.go，将 handler 函数移动到 Handlers.go，且将 Book 结构体移动到 models.go。点击这里[3]查看当前阶段的代码。 现在，我们跳过来写一些测试。在 Go 中，*_test.go 文件是用于测试的。因此让我们创建一个 handlers_test.go。
```
package main

import (
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/julienschmidt/httprouter"
)

func TestBookIndex(t *testing.T) {
    // Create an entry of the book to the bookstore map
    testBook := &Book{
        ISDN:   "111",
        Title:  "test title",
        Author: "test author",
        Pages:  42,
    }
    bookstore["111"] = testBook
    // A request with an existing isdn
    req1, err := http.NewRequest("GET", "/books", nil)
    if err != nil {
        t.Fatal(err)
    }
    rr1 := newRequestRecorder(req1, "GET", "/books", BookIndex)
    if rr1.Code != 200 {
        t.Error("Expected response code to be 200")
    }
    // expected response
    er1 := "{\"meta\":null,\"data\":[{\"isdn\":\"111\",\"title\":\"test title\",\"author\":\"test author\",\"pages\":42}]}\n"
    if rr1.Body.String() != er1 {
        t.Error("Response body does not match")
    }
}

// Mocks a handler and returns a httptest.ResponseRecorder
func newRequestRecorder(req *http.Request, method string, strPath string, fnHandler func(w http.ResponseWriter, r *http.Request, param httprouter.Params)) *httptest.ResponseRecorder {
    router := httprouter.New()
    router.Handle(method, strPath, fnHandler)
    // We create a ResponseRecorder (which satisfies http.ResponseWriter) to record the response.
    rr := httptest.NewRecorder()
    // Our handlers satisfy http.Handler, so we can call their ServeHTTP method
    // directly and pass in our Request and ResponseRecorder.
    router.ServeHTTP(rr, req)
    return rr
}
```
我们使用 httptest 包的 Recorder 来 mock handler。同样，您也可以为 handler BookShow 编写测试。 让我们稍微做些重构。我们仍然把所有路由都定义在了 main 函数中，handler 看起来有点臃肿，我们可以做点 DRY，我们仍然在终端中输出一些日志消息，并且可以添加一个 BookCreate handler 来创建一个新的 Book。 首先，让我们解决 handlers.go。
```
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "io/ioutil"
    "net/http"

    "github.com/julienschmidt/httprouter"
)

func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
    fmt.Fprint(w, "Welcome!\n")
}

// Handler for the books Create action
// POST /books
func BookCreate(w http.ResponseWriter, r *http.Request, params httprouter.Params) {
    book := &Book{}
    if err := populateModelFromHandler(w, r, params, book); err != nil {
        writeErrorResponse(w, http.StatusUnprocessableEntity, "Unprocessible Entity")
        return
    }
    bookstore[book.ISDN] = book
    writeOKResponse(w, book)
}

// Handler for the books index action
// GET /books
func BookIndex(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
    books := []*Book{}
    for _, book := range bookstore {
        books = append(books, book)
    }
    writeOKResponse(w, books)
}

// Handler for the books Show action
// GET /books/:isdn
func BookShow(w http.ResponseWriter, r *http.Request, params httprouter.Params) {
    isdn := params.ByName("isdn")
    book, ok := bookstore[isdn]
    if !ok {
        // No book with the isdn in the url has been found
        writeErrorResponse(w, http.StatusNotFound, "Record Not Found")
        return
    }
    writeOKResponse(w, book)
}

// Writes the response as a standard JSON response with StatusOK
func writeOKResponse(w http.ResponseWriter, m interface{}) {
    w.Header().Set("Content-Type", "application/json; charset=UTF-8")
    w.WriteHeader(http.StatusOK)
    if err := json.NewEncoder(w).Encode(&JsonResponse{Data: m}); err != nil {
        writeErrorResponse(w, http.StatusInternalServerError, "Internal Server Error")
    }
}

// Writes the error response as a Standard API JSON response with a response code
func writeErrorResponse(w http.ResponseWriter, errorCode int, errorMsg string) {
    w.Header().Set("Content-Type", "application/json; charset=UTF-8")
    w.WriteHeader(errorCode)
    json.
        NewEncoder(w).
        Encode(&JsonErrorResponse{Error: &ApiError{Status: errorCode, Title: errorMsg}})
}

//Populates a model from the params in the Handler
func populateModelFromHandler(w http.ResponseWriter, r *http.Request, params httprouter.Params, model interface{}) error {
    body, err := ioutil.ReadAll(io.LimitReader(r.Body, 1048576))
    if err != nil {
        return err
    }
    if err := r.Body.Close(); err != nil {
        return err
    }
    if err := json.Unmarshal(body, model); err != nil {
        return err
    }
    return nil
}
```
我创建了两个函数，writeOKResponse 用于将 StatusOK 写入响应，其返回一个 model 或一个 model slice，writeErrorResponse 将在发生预期或意外错误时将 JSON 错误作为响应。像任何一个优秀的 gopher 一样，我们不应该 panic。我还添加了一个名为 populateModelFromHandler 的函数，它将内容从 body 中解析成所需的任何 model（struct）。在这种情况下，我们在 BookCreate handler 中使用它来填充一个 Book。 现在，我们来看看日志。我们简单地创建一个 Logger 函数，它包装了 handler 函数，并在执行 handler 函数之前和之后打印日志消息。
```
package main

import (
    "log"
    "net/http"
    "time"

    "github.com/julienschmidt/httprouter"
)

// A Logger function which simply wraps the handler function around some log messages
func Logger(fn func(w http.ResponseWriter, r *http.Request, param httprouter.Params)) func(w http.ResponseWriter, r *http.Request, param httprouter.Params) {
    return func(w http.ResponseWriter, r *http.Request, param httprouter.Params) {
        start := time.Now()
        log.Printf("%s %s", r.Method, r.URL.Path)
        fn(w, r, param)
        log.Printf("Done in %v (%s %s)", time.Since(start), r.Method, r.URL.Path)
    }
}
我们来看看路由。首先，在一个地方集中定义所有路由，比如 routes.go。

package main

import "github.com/julienschmidt/httprouter"

/*
Define all the routes here.
A new Route entry passed to the routes slice will be automatically
translated to a handler with the NewRouter() function
*/
type Route struct {
    Name        string
    Method      string
    Path        string
    HandlerFunc httprouter.Handle
}

type Routes []Route

func AllRoutes() Routes {
    routes := Routes{
        Route{"Index", "GET", "/", Index},
        Route{"BookIndex", "GET", "/books", BookIndex},
        Route{"Bookshow", "GET", "/books/:isdn", BookShow},
        Route{"Bookshow", "POST", "/books", BookCreate},
    }
    return routes
}
```
让我们创建一个 NewRouter 函数，它可以在 main 函数中调用，它读取上面定义的所有路由，并返回一个可用的 httprouter.Router。因此创建一个文件 router.go。我们还将使用新创建的 Logger 函数来包装 handler。
```
package main

import "github.com/julienschmidt/httprouter"

//Reads from the routes slice to translate the values to httprouter.Handle
func NewRouter(routes Routes) *httprouter.Router {

    router := httprouter.New()
    for _, route := range routes {
        var handle httprouter.Handle

        handle = route.HandlerFunc
        handle = Logger(handle)

        router.Handle(route.Method, route.Path, handle)
    }

    return router
}
```
您的目录此时应该像这样：
```
.
├── handlers.go
├── handlers_test.go
├── logger.go
├── main.go
├── models.go
├── responses.go
├── router.go
└── routes.go
```
在这里[4]查看完整代码。

这应该可以让你开始编写你自己的 API 服务器了。 你当然需要把你的功能放在不同的包中，所以一个好办法就是：
```
.
├── LICENSE
├── README.md
├── handlers
│   ├── books_test.go
│   └── books.go
├── models
│   ├── book.go
│   └── *
├── store
│   ├── *
└── lib
|   ├── *
├── main.go
├── router.go
├── rotes.go
```
如果您有一个大的单体服务器，您还可以将 handlers、models 和所有路由功能都放在另一个名为 app 的包中。只要记住，go 不像 Java 或 Scala 那样可以有循环的包调用。因此你必须格外小心您的包结构。

这就是全部内容，希望本教程能对您有用。干杯！

注
```
[1] https://github.com/gsingharoy/httprouter-tutorial/tree/master/part1
[2] https://github.com/gsingharoy/httprouter-tutorial/tree/master/part2
[3] https://github.com/gsingharoy/httprouter-tutorial/tree/master/part3
[4] https://github.com/gsingharoy/httprouter-tutorial/tree/master/part4
[Gorilla mux] https://github.com/gorilla/mux
[httprouter] https://github.com/julienschmidt/httprouter
[bone] https://github.com/go-zoo/bone
https://medium.com/@gauravsingharoy/build-your-first-api-server-with-httprouter-in-golang-732b7b01f6ab
```
作者：Gaurav Singha Roy

译者：oopsguy.com

原文: https://studygolang.com/articles/11411