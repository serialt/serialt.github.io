+++
title = 'Go Http'
date = 2023-10-09T20:00:42+08:00
draft = false

tags = ["Go","http"]
categories = ["Go 库文档"]

+++

# Go 原生http库

Go语言内置的`net/http`包十分的优秀，提供了HTTP客户端和服务端的实现。

## 一、net/http介绍

Go语言内置的`net/http`包提供了HTTP客户端和服务端的实现。

#### HTTP协议

超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络传输协议，所有的WWW文件都必须遵守这个标准。设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。





## 二、HTTP客户端

Get、Head、Post和PostForm函数发出HTTP/HTTPS请求。

```go
resp, err := http.Get("http://example.com/")
...
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)
...
resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})
```

程序在使用完response后必须关闭回复的主体。

```go
resp, err := http.Get("http://example.com/")
if err != nil {
	// handle error
}
defer resp.Body.Close()
body, err := ioutil.ReadAll(resp.Body)
// ...
```

### GET请求示例

```go
package main

import (
	"fmt"
	"io"
	"net/http"
)

func main() {
	resp, err := http.Get("https://httpbin.org/uuid")
	if err != nil {
		fmt.Printf("get failed, err:%v\n", err)
		return
	}
	defer resp.Body.Close()
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("read from resp.Body failed, err:%v\n", err)
		return
	}
	fmt.Print(string(body))
}

```

自定义请求

```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"net/url"
)

type UUID struct {
	Uuid string `json:"uuid"`
}

func main() {
	apiURL := "https://httpbin.org/uuid"
	query := url.Values{}
	query.Add("q", "golang")
	query.Add("page", "1")
	ApiURL, _ := url.ParseRequestURI(apiURL)
	ApiURL.RawQuery = query.Encode()

	req, _ := http.NewRequest("GET", ApiURL.String(), nil)
	req.Header.Add("Accept", "*/*")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		return
	}
	defer resp.Body.Close()

	var uuid UUID
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("read from resp.Body failed, err:%v\n", err)
		return
	}
	json.Unmarshal(body, &uuid)
	fmt.Println(uuid.Uuid)
}

```





### POST 请求示例

```go
package main

import (
	"encoding/json"
	"io"
	"net/http"
	"net/url"
)

type dockerToken struct {
	Token string `json:"token"`
}

func main() {

	apiUrl := "https://hub.docker.com/v2/users/login"
	data := url.Values{}
	data.Set("username", "username")
	data.Set("password", "password")
	resp, err := http.PostForm(apiUrl, data)
	if err != nil {
		return
	}
	defer resp.Body.Close()
	_data, _ := io.ReadAll(resp.Body)

	var tmpT dockerToken
	json.Unmarshal(_data, &tmpT)
}

```

```go
package main

import (
	"bytes"
	"io"
	"net/http"
	"net/url"
	"strings"

	"log/slog"
)

func POST1() {
	apiURL := "https://httpbin.org/post"
	form := url.Values{}
	form.Add("ln", "ln222")
	form.Add("ip", "1.1.1.1")
	form.Add("ua", "ua123")

	client := &http.Client{}

	req, _ := http.NewRequest("POST", apiURL, strings.NewReader(form.Encode()))
	req.Header.Set("User-Agent", "test")
	req.Header.Set("Content-Type", "application/x-www-form-urlencoded")

	// 发送请求
	resp, err := client.Do(req)
	if err != nil {
		slog.Error("POST request failed", "err", err)
		return
	}
	defer resp.Body.Close()

	// 读取内容
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		slog.Error("POST request", "err", err)
	} else {
		slog.Info(string(body))
	}

}

func POSTJson() {
	apiURL := "https://httpbin.org/post"

	var jsonStr = []byte(`{"title":"this is a title", "cate": 1}`)

	client := &http.Client{}

	req, _ := http.NewRequest("POST", apiURL, bytes.NewBuffer(jsonStr))
	req.Header.Set("User-Agent", "test")
	req.Header.Set("Content-Type", "application/x-www-form-urlencoded")

	// 发送请求
	resp, err := client.Do(req)
	if err != nil {
		slog.Error("POST request failed", "err", err)
		return
	}
	defer resp.Body.Close()

	// 读取内容
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		slog.Error("POST request", "err", err)
	} else {
		slog.Info(string(body))
	}

}

func main() {
	// POST1()
	POSTJson()
}

```

