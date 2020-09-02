---
title: Iris MVC路由(三)
tags:
  - iris
  - go
keywords:
  - iris
  - go
Categories:
  - Iris
abbrlink: b3c12085
date: 2020-03-03 08:40:28
---

## 定义路由

在根目录 `main.go` 输入以下内容

```go
package main
import (
	"github.com/kataras/iris"
	"github.com/kataras/iris/mvc"
  "./controllers"
)

func main() {
	app := iris.New()
	// 注册视图目录
	app.RegisterView(iris.HTML("./views", ".html"))
	// 定义根路由控制器
	mvc.New(app).Handle(new(controllers.DefaultController))
	// 定义用户分组路由控制器
	mvc.New(app.Party("/user")).Handle(new(controllers.UserController))


	app.Run(iris.Addr(":80"))
}
```

## 创建控制器

创建 `controllers/UserController.go` 输入以下内容

```go
package controllers

import (
	"github.com/kataras/iris"
	"github.com/kataras/iris/mvc"
)

//定义控制器结构体
type UserController struct {
	Ctx iris.Context	//首字母大写-可在外包访问
}
// GetLogin handes GET: http://localhost:80/user/login
// 返回视图-详见第3部分，创建视图
func (uc *UserController) GetLogin() mvc.Result{
	return mvc.View{
		Name:"login.html",
		Data:iris.Map{
			"Title":"登陆",
		},
	}
}
// GetUserinfoBy handles GET: http://localhost:80/user/userinfo/1/2.
// 返回 json 主要看控制器里GET接收参数，以及路由定义规则
// Get为请求方式 驼峰式命名代表路由加 ‘/’ By 关键字接受路由中的参数
func (uc *UserController) GetUserinfoBy(id int,name string) interface{} {
	return iris.Map{
		"id" : id,
		"name" : name,
	}
}

//PostLogin handles POST: http://localhost:80/user/login
//Form 表单 Post 方法接收数据
func (uc *UserController) PostLogin() interface{}{

		username := uc.Ctx.PostValue("username")
		password := uc.Ctx.PostValue("password")

	return iris.Map{
		"username":username,
		"password":password,
	}

}

```



## 创建视图

创建 `view/login.html` 输入以下内容

```html
<head>
    <title>{{.Title}}</title>
</head>
<h1>{{.Title}}</h1>
<form action="login" method="POST">
    <div class="container">
        <label><b>Username</b></label>
        <input type="text" placeholder="Enter Username" name="username" required>

        <label><b>Password</b></label>
        <input type="password" placeholder="Enter Password" name="password" required>

        <button type="submit">Login</button>
    </div>
</form>
```



