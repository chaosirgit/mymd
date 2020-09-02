---
title: Iris 安装及初始(二)
tags:
  - iris
  - go
keywords:
  - iris
  - go
Categories:
  - Iris
abbrlink: cbe049d3
date: 2020-03-01 19:21:06
---

## 安装 

* 要求 GO >= 1.8 推荐 1.9

  ```shell
  go get -u github.com/kataras/iris
  ```

  

## 初始化 Web 服务

```go
package main

import (
	"github.com/kataras/iris"
)

func main()  {
	//创建APP
	app := iris.New()
  //监听端口
	app.Run(iris.Addr(":80"),iris.WithoutServerError(iris.ErrServerClosed))
}
```

## 配置

Iris 支持 `.tml` `.yml` `.json`  3种配置文件。使用 `json` 文件配置需要自定义配置结构，这里我们选用 `.yml` 文件。

`~/gopath/MyApp/config/setting.yml`

```yml
DisablePathCorrection: false
EnablePathEscape: false
FireMethodNotAllowed: true
DisableBodyConsumptionOnUnmarshal: true
TimeFormat: Mon, 01 Jan 2006 15:04:05 GMT
Charset: UTF-8
Other:
	ProjectName: Myiris
```

使用

```go
package main

import (
	"github.com/kataras/iris"
)

func main()  {
	//创建APP
	app := iris.New()
  //监听端口
	app.Run(iris.Addr(":80"),iris.WithConfiguration(iris.YAML("./config/setting.yml")))
}
```

### 自定义配置

#### 定义

在 `settings/config.yml` 输入以下内容

```yml
ProjectName: Myiris
```

#### 封装工具

在 `tools/yamlSetting.go` 输入以下内容

```go
package tools

import (
	"fmt"
	"gopkg.in/yaml.v3"
	"io/ioutil"
)

type YamlSetting struct {
	ProjectName string `yaml:"ProjectName"`
}

func GetSetting(filename string) (s *YamlSetting) {
	configFile, err := ioutil.ReadFile(filename)
	if err != nil {
		//fmt.Printf("parse yaml: %w",err)
		fmt.Errorf("parse yaml: %w", err)
	}

	if err := yaml.Unmarshal(configFile, &s); err != nil {
		fmt.Errorf("parse yaml: %w", err)
		return s
	}
	return s
}

```

#### 使用

```go
package main

import (
	"../tools"
  "fmt"
)



func GetSetting() interface{}  {
	mySetting := tools.GetSetting("./settings/config.yml")
  fmt.Println(mySetting.ProjectName)
}
```

