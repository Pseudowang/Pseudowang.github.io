+++
title = 'Model View Controller'
date = 2025-05-20T19:01:55+08:00
draft = false
categories = ["Notes"]
tags = ["Notes", "Input"]

+++

> 一些粗略地见解，如有写错或低级错误，欢迎您的指正

我可以觉得 Ruby on Rails 中的 MVC 是最有趣的一个技术了 (主要是因为之前并没有了解过 MVC 这个软件架构模式)，一开始有点难懂，后面就慢慢的有趣起来了。

### 什么是 MVC 呢

**MVC** 全称是 Model-View-Controller，是一个软件架构模式，是一种 **软件设计模式**，常用于开发用户界面，它将相关的程序逻辑分为了三个相关联的元素

- **Model**： 信息的内部表达形式。
- **View**： 向用户展示信息并接受用户输入的界面
- **Controller**：连接模型与视图的软件逻辑

### Rails 中的 MVC 架构组织

Rails 框架采用 Model-View-Controller (MVC) 架构组织。通过 MVC，我们有三个主要概念，大部分代码都在其中:

- Model - 管理应用程序中的数据。通常是你的数据库表
- View - 处理以 HTML、JSON、XML 等不同格式呈现的响应
- Controller - 处理用户交互和每个请求的逻辑
  ![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2025/04/e26bdf2f64032d85e5cb309672cdab41.png)

#### Model

是 **MVC** 模式的核心组件。它是应用程序的动态数据结构，独立于用于界面。它可以直接管理应用程序的数据、逻辑和规则。在 Smalltalk-80 中。模型类型的设计完全是由程序员来完成。在 WebObjects、Rails 和 Django 中，模型类型通常代表应用程序数据库中的一个表。就是上面中提到的 Model 在 Rails 中通常是你的数据库表。**Model** 对于保持数据的组织性和一致性至关重要。它可确保应用程序的数据按照已经定义的规则和逻辑运行

```ruby
$ bin/rails generate model Product name:string
```

这行代码就是创建一个 `Product` 模型，然后生成数据库迁移文件，用于创建 `products` 表，并添加 `name` 字段 (类型为 `string`)

`Active Record` 就是 MVC 中的 M 的一部分

#### View

信息的任何表达形式，如图标、示意图或表格。在 Rails 和 Django 中，View 的角色是由 HTML 模板所承担的；Rails 中的 View 是与 Controller Action 是一对一的关系

```ruby
def new
  @topic = Topic.new
end
```

Rails 会 **自动查找并渲染** 对应的视图文件，这个例子就是渲染对应的 `new.html.erb` 文件

Action View 是 MVC 中的 V。 Action Controller 和 Action View 一起处理 web 请求。Action Controller 负责与模型层进行通信并检索数据。然后，Action View 负责使用这些数据渲染响应体以响应 web 请求

#### Controller

接受输入并将其转换为 **Model** 或 **View** 的命令

在 Rails 中，从客户端发往服务端应用的请求首先会被发送到一个"路由器" ( `"router"`)，该路由器会将请求映射到特定控制器的特定方法。在这个方法内部，该控制器会处理请求数据及相关模型对象，并通过视图准备相应内容。通常来说，每个视图都会关联一个对应的控制器。
