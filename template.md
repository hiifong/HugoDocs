---
title: "Hugo Template 文档翻译"
subtitle: ""
date: 2022-10-29T17:11:47+08:00
draft: false
author: "hiifong"
authorLink: "https://i.hiifong.cc"
description: "hiifong的个人博客！"
license: "CC BY-NC 4.0"

tags:
- hugo文档翻译
- hugo doc
categories:
- hugo文档
- hugo文档翻译
- 翻译
---

# [Template](https://gohugo.io/templates/introduction/)



## **基本语法（Basic Syntax）**

Go模板是添加了变量和函数的HTML文件。Go模板的变量和函数可以在`{{ }}`内访问。



### 访问一个预定义的变量(Access a Predefined Variable)

预定义变量可以是已经存在于当前作用域中的变量（像下面变量部分的`.Title`例子），也可以是自定义变量（像同一部分的`$address`例子）。

```shell
{{ .Title }}
{{ $address }}
```

函数的参数用空格分隔。一般的语法是:

```shell
{{ FUNCTION ARG1 ARG2 .. }}
```

下面的例子调用了输入为`1`和`2`的`add`函数。

```shell
{{ add 1 2 }}
```

### 方法和字段通过点状符号进行访问(Methods and Fields are Accessed via dot Notation) 

访问在一块内容的前言中定义的页面参数栏。

```shell
{{ .Params.bar }}
```

### 括号可用于将项目组合在一起(Parentheses Can be Used to Group Items Together)

```shell
{{ if or (isset .Params "alt") (isset .Params "caption") }} Caption {{ end }}
```

### 一条语句可以分割成多行(A Single Statement Can be Split over Multiple Lines)

```shell
{{ if or
  (isset .Params "alt")
  (isset .Params "caption")
}}
```

### 原始字符串字面值可以包括换行符(Raw String Literals Can Include Newlines)

```shell
{{ $msg := `Line one.
Line two.` }}
```

### 变量(Variables) 

每个Go模板都会得到一个数据对象。在Hugo中，每个模板都会传递一个`Page`。在下面的例子中，`.Title`是该Page变量中可访问的元素之一。

由于Page是模板的默认范围，当前范围内的Title元素（`.`-"点"）可以通过点状前缀（`.Title`）来访问。

```html
<title>{{ .Title }}</title>
```

值也可以存储在自定义变量中，并在以后引用。

>   自定义变量需要以`$`为前缀。

```shell
{{ $address := "123 Main St." }}
{{ $address }}
```

变量可以使用`=`操作符重新定义。下面的例子在主页上打印 "Var is Hugo Home"，而在所有其他页面上打印 "Var is Hugo Page":

```shell
{{ $var := "Hugo Page" }}
{{ if .IsHome }}
    {{ $var = "Hugo Home" }}
{{ end }}
Var is {{ $var }}
```



### 函数(Functions)

Go模板只提供了一些基本的函数，但也为应用程序提供了一个机制来扩展原有的函数集。

Hugo模板函数提供了建立网站所需的额外函数。函数的调用是通过使用它们的名字，后面是用空格分隔的所需参数。如果不重新编译Hugo，就不能添加模板函数。

#### 例1：数字加法(Adding Numbers)

```shell
{{ add 1 2 }}
<!-- prints 3 -->
```

#### 例2：数字的比较(Comparing Numbers)

```shell
{{ lt 1 2 }}
<!-- prints true (i.e., since 1 is less than 2) -->
```

请注意，这两个例子都是利用了Go模板的数学函数。

>   Go模板文档中的布尔运算符比Hugo文档中列出的更多。

### 包括(Includes) 

当包括另一个模板时，你需要把它需要访问的数据传递给它。

为了传递当前的上下文，请记住包含一个尾部的点(`.`)。

模板的位置将总是从Hugo中的`layouts/`目录开始。

### 局部(Partial) 

部分函数用于包含部分模板，语法是`{{ partial "<PATH>/<PARTIAL>.<EXTENSION>" . }}`

可用的内部模板可以在[这里]()找到。

#### 包括内部opengraph.html模板的例子。

```shell
{{ template "_internal/opengraph.html" . }}
```

### 逻辑(Logic)

Go模板提供最基本的迭代和条件逻辑。

#### 迭代(Iteration) 

Go模板大量使用`range`来迭代map、数组或切片。以下是如何使用`range`的不同例子。

##### 例1：使用上下文（`.`） 

```shell
{{ range $array }}
    {{ . }} <!-- The . represents an element in $array -->
{{ end }}
```

##### 例2: 为一个数组元素的值声明一个变量名

```shell
{{ range $elem_val := $array }}
    {{ $elem_val }}
{{ end }}
```

##### 例3：为数组元素的索引和值声明变量名 

对于一个数组或切片, 第一个声明的变量将映射到每个元素的索引.

```shell
{{ range $elem_index, $elem_val := $array }}
   {{ $elem_index }} -- {{ $elem_val }}
{{ end }}
```

##### 例4：为一个`map`元素的键和值声明变量名 

对于一个`map`，第一个声明的变量将映射到每个`map`元素的键。

```shell
{{ range $elem_key, $elem_val := $map }}
   {{ $elem_key }} -- {{ $elem_val }}
{{ end }}
```

##### 例子5：对空`map`、数组或切片的条件限制 

如果传入`range`的`map`、`array`或`slice`是长度为0，那么else语句就会被执行。

```shell
{{ range $array }}
    {{ . }}
{{else}}
    <!-- This is only evaluated if $array is empty -->
{{ end }}
```

### 条件式(Conditionals) 

`if`、`else`、`with`、`or`、`and` 和 `not`提供了Go模板中处理条件逻辑的框架。和`range`一样，`if`和`with`语句也是用`{{ end }}`来结束的。

Go模板将以下值视为`false`:

-   false (布尔值)
-   0 (整数)
-   任何零长度的数组、切片、map或字符串

##### 例1：`with`

使用`with`编写 "如果有东西存在，就这样做 "的语句是很常见的。

>   `with`在其范围内重新绑定上下文.（就像在`range`中）。

如果变量不存在，或者像上面解释的那样评估为 "false"，它就跳过这个块。

```html
{{ with .Params.title }}
    <h4>{{ . }}</h4>
{{ end }}
```

##### 例2：`with ... else`

下面的片段使用 "description "前面的参数值，如果设置了，否则使用默认的`.Summary` Page变量。

```shell
{{ with .Param "description" }}
    {{ . }}
{{ else }}
    {{ .Summary }}
{{ end }}
```

>   See the [`.Param` function](https://gohugo.io/functions/param/).

##### 例3：`if`

另一种写`with`的方式（也是一种更冗长的方式）是使用`if`。在这里，`.`不会被反弹。

下面的例子是用`if`重写的 "例子1"。

```shell
{{ if isset .Params "title" }}
    <h4>{{ index .Params "title" }}</h4>
{{ end }}
```

##### 例4：`if ... else`

下面的例子是用`if ... else`重写的 "例子2"，并使用`isset`函数加`.Params`变量（与`.Param`函数不同）代替。

```shell
{{ if (isset .Params "description") }}
    {{ index .Params "description" }}
{{ else }}
    {{ .Summary }}
{{ end }}
```

##### 例5：`if ... else if ... else` 

与`with`不同，`if`也可以包含`else if`子句。

```shell
{{ if (isset .Params "description") }}
    {{ index .Params "description" }}
{{ else if (isset .Params "summary") }}
    {{ index .Params "summary" }}
{{ else }}
    {{ .Summary }}
{{ end }}
```

##### 例6：`and`和`or`

```shell
{{ if (and (or (isset .Params "title") (isset .Params "caption")) (isset .Params "attr")) }}
```

###  管道（`Pipes`)

Go模板最强大的组件之一是能够一个接一个地堆叠动作。这是通过使用管道来实现的。借用了Unix管道的概念，这个概念很简单：每个管道的输出都成为下一个管道的输入。

由于Go模板的语法非常简单，管道是能够将函数调用连锁起来的关键。管道的一个限制是，它们只能处理一个值，而且这个值会成为下一个管道的最后一个参数。

几个简单的例子应该有助于传达如何使用管道。

##### 例子1：`shuffle` 

下面的两个例子在功能上是一样的。

```shell
{{ shuffle (seq 1 5) }}
```

```shell
{{ (seq 1 5) | shuffle }}
```

##### 例2：索引

以下是访问名为 "disqus_url "的页面参数，并对HTML进行转义。这个例子也使用了`index`函数，它是Go模板中内置的。

```shell
{{ index .Params "disqus_url" | html }}
```

##### 例3：`or`和`isset`

```shell
{{ if or (or (isset .Params "title") (isset .Params "caption")) (isset .Params "attr") }}
Stuff Here
{{ end }}
```

可以改写为:

```shell
{{ if isset .Params "caption" | or isset .Params "title" | or isset .Params "attr" }}
Stuff Here
{{ end }}
```

### 上下文（又称 `.`）。

关于Go模板，最容易被忽视的概念是：`{{ . }}`总是指的是当前的上下文。

-   在你的模板的顶层，这将是向它提供的数据集。
-   然而，在一个迭代中，它将具有循环中的当前项目的值；也就是说，`{{ . }}`将不再是指整个页面的可用数据。

如果你需要在循环中访问页面级别的数据（例如，在前面的事项中设置的页面参数），你可能想做以下事情之一。

#### 1.定义一个独立于上下文的变量 

下面展示了如何定义一个独立于上下文的变量。

```shell
# tags-range-with-page-variable.html

{{ $title := .Site.Title }}
<ul>
{{ range .Params.tags }}
    <li>
        <a href="/tags/{{ . | urlize }}">{{ . }}</a>
        - {{ $title }}
    </li>
{{ end }}
</ul>
```

>   请注意，一旦我们进入循环（即 `range`），`{{ . }}`的值已经改变。我们在循环外定义了一个变量（`{{$title}}`），并为其赋值，这样我们在循环内也可以访问该值。

#### 2.使用`$`来访问全局上下文 

`$`在你的模板中具有特殊的意义。默认情况下，`$`被设置为`.`的起始值。这是Go `text/template`的一个有记录的特征。这意味着你可以从任何地方访问全局上下文。下面是前面代码块的一个等效例子，但现在使用`$`从全局上下文中获取`.Site.Title`。

```shell
# range-through-tags-w-global.html


<ul>
{{ range .Params.tags }}
  <li>
    <a href="/tags/{{ . | urlize }}">{{ . }}</a>
            - {{ $.Site.Title }}
  </li>
{{ end }}
</ul>
```

>   如果有人恶作剧地重新定义这个特殊字符，`$`的内置魔力将不再起作用；例如，`{{ $ := .Site }}`。请不要这样做。当然，你可以通过在全局范围内使用`{{ $ := .}}`将`$`重置为其默认值来恢复这种恶作剧。

### 留白(`Whitespace`) 

Go 1.6 包含了修剪 Go 标签两侧空白的功能，即在相应的 `{{ `或 `}} `分隔符旁边加上连字符（`-`）和空格。

例如，下面的Go模板将在其HTML输出中包括换行符和水平制表符。

```html
<div>
  {{ .Title }}
</div>
```

这将输出:

```html
<div>
  Hello, World!
</div>
```

在下面的例子中，利用`-`将删除`.Title`变量周围多余的空白部分，并删除换行。

```html
<div>
  {{- .Title -}}
</div>
```

这将输出:

```html
<div>Hello, World!</div>
```

Go认为以下字符为空白。

-   空格
-   水平制表符
-   回车
-   换行

### 注释（Comments)

为了使你的模板有条不紊，并在整个团队中分享信息，你可能想给你的模板添加注释。使用Hugo有两种方法可以做到这一点。

#### Go模板的注释

Go模板支持`{{/*`和`*/}}`来打开和关闭一个评论块。该块中的任何内容都不会被渲染。

比如:

```shell
Bonsoir, {{/* {{ add 0 + 2 }} */}}Eliott.
```

将呈现`Bonsoir, Eliott.`，并不关心注释区的语法错误（`add 0 + 2`）。

#### HTML注释 

如果你需要从你的模板中产生HTML注释，可以看看Internet Explorer的条件注释例子。如果你需要变量来构建这样的HTML注释，只需用管道将`printf`送到`safeHTML`。比如:

```shell
{{ printf "<!-- Our website is named: %s -->" .Site.Title | safeHTML }}
```

#### HTML注释含有go模板 

HTML注释默认是被剥离的，但其内容仍然被评估。这意味着，尽管HTML注释不会向最终的HTML页面呈现任何内容，但注释中包含的代码可能会使构建过程失败。

>   不要试图用HTML注释来注释Go模板的代码。

```shell
<!-- {{ $author := "Emma Goldman" }} was a great woman. -->
{{ $author }}
```

模板引擎将剥离HTML注释中的内容，但会首先评估其中存在的任何Go模板代码。所以上面的例子会呈现`Emma Goldman`，因为`$author`变量在HTML注释中被评估了。但如果HTML注释中的代码有错误，构建就会失败。

### Hugo参数(Hugo Parameters) 

Hugo提供了一个选项，即通过你的网站配置（即网站范围内的值）或通过每个特定内容的元数据（即正面内容）将值传递给你的模板层。你可以定义任何类型的值，并在你的模板中随意使用它们，只要这些值被前面格式所支持。

### 使用内容（`Page`）参数 

你可以在个别内容的前言中提供被模板使用的变量。

这方面的一个例子是用在Hugo的文档中。大多数页面都受益于提供的目录，但有时目录并没有很大的意义。我们在前言中定义了一个`notoc`变量，当具体设置为 `true`时，将阻止目录的呈现。

下面是例子（YAML）。

```yaml
---
title: Roadmap
lastmod: 2017-03-05
date: 2013-11-18
notoc: true
---
```

下面是一个可以在`toc.html`部分模板内使用的相应代码的例子:

```html
<!-- layouts/partials/toc.html -->
{{ if not .Params.notoc }}
<aside>
  <header>
    <a href="#{{.Title | urlize}}">
    <h3>{{.Title}}</h3>
    </a>
  </header>
  {{.TableOfContents}}
</aside>
<a href="#" id="toc-toggle"></a>
{{ end }}
```

我们希望页面的默认行为是包括一个`TOC`，除非另有规定。这个模板检查以确保这个页面的正面内容中的`notoc`:字段是不是`true`。

### 使用网站配置参数(Use Site Configuration Parameters) 

你可以在你的网站的配置文件中任意定义任意多的网站级参数。这些参数在你的模板中是全局可用的。

例如，你可以声明以下内容。

-   config.yaml

```yaml
params:
  copyrighthtml: Copyright &#xA9; 2017 John Doe. All Rights Reserved.
  sidebarrecentlimit: 5
  twitteruser: spf13
```

-   config.toml

```toml
[params]
  copyrighthtml = 'Copyright &#xA9; 2017 John Doe. All Rights Reserved.'
  sidebarrecentlimit = 5
  twitteruser = 'spf13'
```

-   config.json

```json
{
   "params": {
      "copyrighthtml": "Copyright \u0026#xA9; 2017 John Doe. All Rights Reserved.",
      "sidebarrecentlimit": 5,
      "twitteruser": "spf13"
   }
}
```

在一个页脚布局中，你可以声明一个`<footer>`，只有在提供了`copyrighthtml`参数时才会被渲染。如果提供了这个参数，你就需要通过 `safeHTML` 函数声明这个字符串是安全的，这样HTML实体就不会再被转义。这将让你在每年1月1日轻松地只更新你的顶层配置文件，而不是在你的模板中寻找。

```html
{{ if .Site.Params.copyrighthtml }}
    <footer>
        <div class="text-center">{{.Site.Params.CopyrightHTML | safeHTML}}</div>
    </footer>
{{ end }}
```

另一种写 `if `然后引用相同值的方法是用`with`代替。`with`在其范围内重新绑定上下文（`.`），如果变量不存在，则跳过该块。

```html
<!-- layouts/partials/twitter.html -->

{{ with .Site.Params.twitteruser }}
    <div>
        <a href="https://twitter.com/{{.}}" rel="author">
        <img src="/images/twitter.png" width="48" height="48" title="Twitter: {{.}}" alt="Twitter"></a>
    </div>
{{ end }}
```

最后，你也可以从你的布局中拉出 "神奇的常数"。下面使用了`first`函数，以及`.RelPermalink`页面变量和`.Site.Pages`站点变量。

```html
<nav>
  <h1>Recent Posts</h1>
  <ul>
  {{- range first .Site.Params.SidebarRecentLimit .Site.Pages -}}
      <li><a href="{{.RelPermalink}}">{{.Title}}</a></li>
  {{- end -}}
  </ul>
</nav>
```

##### 例子: 显示未来事件(Show Future Events)

给出以下内容结构和正面内容。

```shell
content/
└── events/
    ├── event-1.md
    ├── event-2.md
    └── event-3.md
```

`content/events/event-1.md`的配置信息

```yaml
---
date: 2021-12-06T10:37:16-08:00
draft: false
end_date: 2021-12-05T11:00:00-08:00
start_date: 2021-12-05T09:00:00-08:00
title: Event 1
---
```

这个部分模板渲染了未来的事件:

```html
<!-- layouts/partials/future-events.html -->

<h2>Future Events</h2>
<ul>
  {{ range where site.RegularPages "Type" "events" }}
    {{ if gt (.Params.start_date | time.AsTime) now }}
      {{ $startDate := .Params.start_date | time.Format ":date_medium" }}
      <li>
        <a href="{{ .RelPermalink }}">{{ .LinkTitle }}</a> - {{ $startDate }}
      </li>
    {{ end }}
  {{ end }}
</ul>
```

如果你把前面的内容限制为TOML格式，并且省略日期字段周围的引号，你就可以不通过铸造来进行日期比较。

```html
<!-- layouts/partials/future-events.html -->

<h2>Future Events</h2>
<ul>
  {{ range where (where site.RegularPages "Type" "events") "Params.start_date" "gt" now }}
    {{ $startDate := .Params.start_date | time.Format ":date_medium" }}
    <li>
      <a href="{{ .RelPermalink }}">{{ .LinkTitle }}</a> - {{ $startDate }}
    </li>
  {{ end }}
</ul>
```

