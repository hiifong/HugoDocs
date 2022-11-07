# [函数](https://gohugo.io/functions/)

Go模板是轻量级的，但可扩展的。Go本身提供了内置函数，包括比较运算符和其他基本工具。这些都列在Go模板文档中。Hugo为基本的模板逻辑添加了额外的函数。

# `.AddDate`

返回将给定的年、月、日数添加到给定的time.Time值中的相应时间。
**语法:**

```shell
.AddDate YEARS MONTHS DAYS
```

```shell
{{ $d := "2022-01-01" | time.AsTime }}

{{ $d.AddDate 0 0 1 | time.Format "2006-01-02" }} --> 2022-01-02
{{ $d.AddDate 0 1 1 | time.Format "2006-01-02" }} --> 2022-02-02
{{ $d.AddDate 1 1 1 | time.Format "2006-01-02" }} --> 2023-02-02

{{ $d.AddDate -1 -1 -1 | time.Format "2006-01-02" }} --> 2020-11-30
```

>   当添加月份或年份时，如果产生的日期不存在，Hugo会将最终的time.Time值标准化。例如，在1月31日的基础上增加一个月，产生3月2日或3月3日，这取决于年份。
>
>   请看Go团队的[解释](https://github.com/golang/go/issues/31145#issuecomment-479067967)。

```shell
{{ $d := "2023-01-31" | time.AsTime }}
{{ $d.AddDate 0 1 0 | time.Format "2006-01-02" }} --> 2023-03-03

{{ $d := "2024-01-31" | time.AsTime }}
{{ $d.AddDate 0 1 0 | time.Format "2006-01-02" }} --> 2024-03-02

{{ $d := "2024-02-29" | time.AsTime }}
{{ $d.AddDate 1 0 0 | time.Format "2006-01-02" }} --> 2025-03-01
```

# `.Format`

根据Go的布局字符串对内置的hugo日期----`.Date`、`.PublishDate`和`.Lastmod`----进行格式化。
语法:

```shell
.Format FORMAT
```

`.Format`将格式化在你的前言中定义的日期值，并可作为以下页面变量的属性使用。

-   `.PublishDate`
-   `.Date`

-   `.Lastmod`
    假设在内容文件的前言中有`date: 2017-03-03`，你可以在构建时通过`.Format`和一个布局字符串来运行你所需要的输出。

```shell
{{ .PublishDate.Format "January 2, 2006" }} => March 3, 2017
```

对于在你的前言中定义的任何日期的字符串表示的格式化，请参阅`dateFormat`函数，它仍将利用下面解释的Go布局字符串，但使用的语法略有不同。

## Go的布局字符串 

Hugo模板通过指向特定参考时间的布局字符串格式化你的日期。

```shell
Mon Jan 2 15:04:05 MST 2006
```

虽然这看起来很随意，但`MST`的数值是`07`，从而使布局字符串成为一串数字。

[这里有一个直接来自Go文档的直观解释](https://golang.org/pkg/time/#example_Time_Format)。

```shell
 Jan 2 15:04:05 2006 MST
=> 1 2  3  4  5    6  -7
```

## hugo日期和时间模板参考 

下面的例子显示了布局字符串，然后是渲染的输出。

这些例子是在CST中渲染和测试的，并且都指向内容文件中的同一个字段。

```shell
date: 2017-03-03T14:15:59-06:00
```

`.Date` (i.e. called via [page variable](https://gohugo.io/variables/page/))

**Returns**: `2017-03-03 14:15:59 -0600 CST`

```shell
"Monday, January 2, 2006"
```

**Returns**: `Friday, March 3, 2017`

```shell
"Mon Jan 2 2006"
```

**Returns**: `Fri Mar 3 2017`

```shell
"January 2006"
```

**Returns**: `March 2017`

```shell
"2006-01-02"
```

**Returns**: `2017-03-03`

```shell
"Monday"
```

**Returns**: `Friday`

`"02 Jan 06 15:04 MST"` (RFC822)

**Returns**: `03 Mar 17 14:15 CST`

`"02 Jan 06 15:04 -0700"` (RFC822Z)

**Returns**: `03 Mar 17 14:15 -0600`

`"Mon, 02 Jan 2006 15:04:05 MST"` (RFC1123)

**Returns**: `Fri, 03 Mar 2017 14:15:59 CST`

`"Mon, 02 Jan 2006 15:04:05 -0700"` (RFC1123Z)

**Returns**: `Fri, 03 Mar 2017 14:15:59 -0600`

更多的例子可以在Go的时间包的文档中找到。

## 红心数和序数的缩写

目前不支持拼出的心数（如 "1"、"2"和 "3"）。

目前不直接支持序数缩略语（即短后缀，如 "1st"、"2nd"和 "3rd"）。通过使用`{{.Date.Format "Jan 2nd 2006"}}`，Hugo假定你想将`nd`作为字符串附加到月份的日期上。不过，你可以把函数串联起来，形成这样的结果。

```shell
{{ .Date.Format "2" }}{{ if in (slice 1 21 31) .Date.Day}}st{{ else if in (slice 2 22) .Date.Day}}nd{{ else if in (slice 3 23) .Date.Day}}rd{{ else }}th{{ end }} of {{ .Date.Format "January 2006" }}
```

这将输出:

```shell
5th of March 2017
```

## 使用`.Local`和`.UTC`

结合`dateFormat`函数，你也可以将你的日期转换为UTC或本地时区。

```shell
{{ dateFormat "02 Jan 06 15:04 MST" .Date.UTC }}
```

**Returns**: `03 Mar 17 20:15 UTC`

```shell
{{ dateFormat "02 Jan 06 15:04 MST" .Date.Local }}
```

**Returns**: `03 Mar 17 14:15 CST`

# `.Get`

访问`shortcode`声明中的位置和顺序参数。

语法：

```shell
.Get INDEX
```

```shell
.Get KEY
```

`.Get`是在创建你自己的`shortcode`模板时专门使用的，用来访问传递给它的位置和命名参数。当使用一个数字INDEX时，它查询位置参数（从0开始）。与一个字符串KEY一起使用，它查询命名参数。

当访问一个不存在的命名参数时，`.Get`返回一个空字符串，而不是中断构建。在hugo 0.40及以后的版本中，位置参数也是如此。这允许你将`.Get`与`if`、`with`、`default`或`cond`链接，以检查参数是否存在。例如，你现在可以使用。

```shell
{{ $quality := default "100" (.Get 1) }}
```

# `.GetPage`

获取给定路径的页面。

语法：

```shell
.GetPage PATH
```

`.GetPage`返回一个给定路径的页面。`Site`和`Page`都实现了这个方法。如果给定的是一个相对路径---即没有前导符的路径---`Page`变体将尝试寻找相对于当前页面的页面。

>   注意：我们在Hugo 0.45中全面简化了`.GetPage` API。在此之前，除了路径之外，你还需要提供一个Kind属性，例如`{{ .Site.GetPage "section" "blog" }}`。这仍然可以工作，但现在是多余的了。

```shell
{{ with .Site.GetPage "/blog" }}{{ .Title }}{{ end }}
```

当找不到页面时，这个方法将返回`nil`，所以如果没有找到博客部分，上述方法将不会打印任何东西。

要在博客部分找到一个普通的页面：

```shell
{{ with .Site.GetPage "/blog/my-post.md" }}{{ .Title }}{{ end }}
```

而由于`Page`也提供了一个`.GetPage`方法，所以上面的写法与下面的写法一致：

```shell
{{ with .Site.GetPage "/blog" }}
{{ with .GetPage "my-post.md" }}{{ .Title }}{{ end }}
{{ end }}
```

#### `.GetPage`和多语言网站 

前面的例子都是用完整的内容文件名来查找帖子的。根据你组织内容的方式（你是否在文件名中加入了语言代码，例如`my-post.en.md`），你可能想做不带扩展名的查询。这将使你得到该网页的当前语言的版本。

```shell
{{ with .Site.GetPage "/blog/my-post" }}{{ .Title }}{{ end }}
```

#### `.GetPage` 示例 

这个代码片断--以部分模板的形式--允许你做以下事情。

1.  抓取你的标签分类法的索引对象。
2.  将此对象分配给一个变量，`$t`
3.  按流行程度对与分类法相关的术语进行排序。
4.  抓取分类法中最受欢迎的两个术语（即分配给内容的两个最受欢迎的标签。

```html
<!-- grab-top-two-tags.html -->

<ul class="most-popular-tags">
{{ $t := .Site.GetPage "/tags" }}
{{ range first 2 $t.Data.Terms.ByCount }}
    <li>{{ . }}</li>
{{ end }}
</ul>
```

#### `.GetPage` on Page Bundle 

如果由`.GetPage`检索的页面是一个[Leaf Bundle](https://gohugo.io/content-management/page-bundles/#leaf-bundles)，并且你需要获得其中的嵌套页面资源，你将需要使用`.Resources`中的方法，如[页面资源](https://gohugo.io/content-management/page-resources/)部分所解释的。

见[Headless Bundle](https://gohugo.io/content-management/page-bundles/#headless-bundle)文档中的例子。

# `.HasMenuCurrent`

语法：

```shell
PAGE.HasMenuCurrent MENU MENUENTRY
```

`.HasMenuCurrent`是`Page`对象的一个方法，返回一个布尔值。如果该PAGE与给定菜单中MENUENTRY下的一个子菜单条目中的`.Page`是同一个对象，它将返回true。

`v0.86.0`中的新内容。如果MENUENTRY的`.Page`是一个部分，那么从`Hugo 0.86.0`开始，这个方法对该部分的任何后裔也返回真。

你可以在[菜单模板](https://gohugo.io/templates/menu-templates/)中找到它的使用实例。

# `.IsMenuCurrent`

语法：

```shell
PAGE.IsMenuCurrent MENU MENUENTRY
```

`.IsMenuCurrent`是`Page`对象中的一个方法，返回一个布尔值。如果PAGE与给定菜单中MENUENTRY中的`.Page`是同一个对象，它返回真。

你可以在[菜单模板](https://gohugo.io/templates/menu-templates/)中找到它的使用实例。

# `.Param`

在你的模板中调用页面或网站变量。

语法：

```shell
.Param KEY
```

在Hugo中，你可以声明全站的参数（即在你的配置中），也可以声明单个页面的参数。

一个常见的用例是为网站设定一个一般的值，为某些页面设定一个更具体的值（例如，一个图片）。

你可以使用`.Param`方法来调用这些值到你的模板中。下面将首先在特定内容的前言中寻找一个图像参数。如果没有找到，Hugo将在你的网站配置中寻找一个图像参数。

```shell
$.Param "image"
```

>   `Param`方法可能不会把内容前言的空字符串视为 "未找到"。如果你使用Hugo的原型将预设的前述字段设置为空字符串，最好使用`default`函数而不是`Param`。参见GitHub上的相关问题。

# `.Render`

取一个视图，在渲染内容时应用。

语法：

```shell
.Render LAYOUT
```

视图是一个替代的布局，应该是一个文件名，指向[内容视图](https://gohugo.io/templates/views)文档中指定的一个位置的模板。

这个功能只有在应用于[列表上下文](https://gohugo.io/templates/lists/)中的单一内容时才可用。

这个例子可以使用位于`/layouts/_default/summary.html`的内容视图来渲染一段内容。

```shell
{{ range .Pages }}
    {{ .Render "summary"}}
{{ end }}
```

# `.RenderString`

渲染标记为HTML。

语法：

```shell
.RenderString MARKUP
```

`0.62.0`版的新内容
`.RenderString`是`Page`上的一个方法，它使用为该页面定义的内容渲染器（如果没有在选项中设置）将一些标记渲染成HTML。

该方法需要一个带有这些选项的可选地图参数:

**`display ("inline")`** 
`inline`或`block`。如果是`inline`（默认），短文段周围的`<p></p>`将被修剪。

**`markup (默认为页面的标记)`**
见内容格式列表中的标识符。

一些例子:

```shell
{{ $optBlock := dict "display" "block" }}
{{ $optOrg := dict "markup" "org" }}
{{ "**Bold Markdown**" | $p.RenderString }}
{{  "**Bold Block Markdown**" | $p.RenderString  $optBlock }}
{{  "/italic org mode/" | $p.RenderString  $optOrg }}
```

`v0.93.0`中的新功能

注意：`markdownify`使用这个功能是为了支持`Render Hooks`。

# `.Scratch`

作为一个 "scratchpad "来存储和处理数据。
Scratch是Hugo的一项功能，旨在方便地在Go模板世界中操作数据。它既可以是一个页面或简码方法，所产生的数据将被附加到给定的上下文中，也可以作为一个独特的实例存储在一个变量中。

>   请注意，`Scratch`最初是为了解决影响0.48之前的Hugo版本的变通办法模板范围限制而创建的。关于`.Scratch`和上下文用例的详细分析，请参阅[这篇博文](https://regisphilibert.com/blog/2017/04/hugo-scratch-explained-variable/)。

上下文的`.Scratch`与本地的`newScratch`对比

从Hugo 0.43开始，有两种不同的方式来使用Scratch。

## 页面的`.Scratch`

`.Scratch`可以作为一个页面方法或一个`Shortcode`方法使用，并将 "scratched "数据附加到给定的页面。使用`.Scratch`需要一个页面或一个`Shortcode`上下文。

```shell
{{ .Scratch.Set "greeting" "bonjour" }}
{{ range .Pages }}
  {{ .Scratch.Set "greeting" (print "bonjour" .Title) }}
{{ end }}
```

## 本地的`newScratch`

`v0.43`中的新内容

一个Scratch实例也可以使用`newScratch`函数分配给任何变量。在这种情况下，不需要页面或`Shortcode`的上下文，而且`scratch`的范围也只是本地的。下面详述的方法可以从Scratch实例被分配到的变量中获得。

```shell
{{ $data := newScratch }}
{{ $data.Set "greeting" "hola" }}
```

## 方法

一个Scratch有以下方法。

请注意，下面的例子假设一个本地的Scratch实例已经存储在`$scratch`中。

### `.Set`方法

设置一个给定的键的值。

```shell
{{ $scratch.Set "greeting" "Hello" }}
```

### `.Get`方法

获取一个给定的键的值。

```shell
{{ $scratch.Set "greeting" "Hello" }}
----
{{ $scratch.Get "greeting" }} > Hello
```

### `.Add`方法

将一个给定的值添加到给定键的现有值中。

对于单个值，`Add`接受支持Go的`+`运算符的值。如果一个键的第一次添加是一个数组或切片，下面的添加将被附加到该列表中。

```shell
{{ $scratch.Add "greetings" "Hello" }}
{{ $scratch.Add "greetings" "Welcome" }}
----
{{ $scratch.Get "greetings" }} > HelloWelcome
```

```shell
{{ $scratch.Add "total" 3 }}
{{ $scratch.Add "total" 7 }}
----
{{ $scratch.Get "total" }} > 10
```

```shell
{{ $scratch.Add "greetings" (slice "Hello") }}
{{ $scratch.Add "greetings" (slice "Welcome" "Cheers") }}
----
{{ $scratch.Get "greetings" }} > []interface {}{"Hello", "Welcome", "Cheers"}
```

### `.SetInMap`方法

接受一个`key`，`mapKey`和`value`，并将`mapKey`和`value`的map添加到给定的`key`中。

```shell
{{ $scratch.SetInMap "greetings" "english" "Hello" }}
{{ $scratch.SetInMap "greetings" "french" "Bonjour" }}
----
{{ $scratch.Get "greetings" }} > map[french:Bonjour english:Hello]
```

### `.DeleteInMap`方法

接受一个`key`和`mapKey`，从给定的`key`中删除`mapKey`的map。

```shell
{{ .Scratch.SetInMap "greetings" "english" "Hello" }}
{{ .Scratch.SetInMap "greetings" "french" "Bonjour" }}
----
{{ .Scratch.DeleteInMap "greetings" "english" }}
----
{{ .Scratch.Get "greetings" }} > map[french:Bonjour]
```

### `.GetSortedMapValues`方法

返回一个按`mapKey`排序的`key`的值数组。

```shell
{{ $scratch.SetInMap "greetings" "english" "Hello" }}
{{ $scratch.SetInMap "greetings" "french" "Bonjour" }}
----
{{ $scratch.GetSortedMapValues "greetings" }} > [Hello Bonjour]
```

### `.Delete`方法

`v0.38`中的新功能

删除给定的键。

### `.Values`方法

返回原始的支持map。注意，你应该只在你通过`newScratch`获得的本地范围的Scratch实例上使用这个方法，而不是`.Page.Scratch`等，因为这将导致并发问题。

# `.Unix`

将`time.Time`值转换为自Unix纪元起经过的秒数，不包括闰秒。Unix的纪元是1970年1月1日的00:00:00 UTC。

语法：

```shell
.Unix

.UnixMilli

.UnixMicro

.UnixNano
```

`Milli`、`Micro`和`Nano`变量返回自Unix epoch以来经过的毫秒、微秒和纳秒的数量（分别）。

```shell
.Date.Unix        --> 1637259694
.ExpiryDate.Unix  --> 1672559999
.Lastmod.Unix     --> 1637361786
.PublishDate.Unix --> 1637421261

("1970-01-01T00:00:00-00:00" | time.AsTime).Unix --> 0
("1970-01-01T00:00:42-00:00" | time.AsTime).Unix --> 42
("1970-04-11T01:48:29-08:00" | time.AsTime).Unix --> 8675309
("2026-05-02T20:09:31-07:00" | time.AsTime).Unix --> 1777777771

now.Unix      --> 1637447841
now.UnixMilli --> 1637447841347
now.UnixMicro --> 1637447841347378
now.UnixNano  --> 1637447841347378799
```

# `absLangURL`

根据多语言网站的配置，添加带有正确语言前缀的绝对URL。

语法：

```shell
absLangURL INPUT
```

`absLangURL`和`relLangURL`都类似于它们的`absURL`和`relURL`的亲戚，但是当网站配置了不止一种语言时，会添加正确的语言前缀。

因此，对于一个网站的baseURL设置为https://example.com/hugo/，并且当前的语言是en。

```shell
{{ "blog/" | absLangURL }} → "https://example.com/hugo/en/blog/"
{{ "blog/" | relLangURL }} → "/hugo/en/blog/"
```

# `absURL`

基于配置的`baseURL`，创建一个绝对的URL。

语法：

```shell
absURL INPUT
```

`absURL`和`relURL`都考虑了你的网站配置文件中的`baseURL`的配置值。给定一个baseURL设置为https://example.com/hugo/。

```shell
{{ "mystyle.css" | absURL }} → "https://example.com/hugo/mystyle.css"
{{ "mystyle.css" | relURL }} → "/hugo/mystyle.css"
{{ "http://gohugo.io/" | relURL }} →  "http://gohugo.io/"
{{ "http://gohugo.io/" | absURL }} →  "http://gohugo.io/"
```

最后两个例子可能看起来很奇怪，但可能非常有用。例如，下面显示了如何在JSON-LD结构化数据（SEO）中使用`absURL`，在这种情况下，你的一段内容的一些图片可能会或可能不会被托管在本地。

```html
<!-- layouts/partials/schemaorg-metadata.html -->

<script type="application/ld+json">
{
    "@context" : "http://schema.org",
    "@type" : "BlogPosting",
    "image" : {{ apply .Params.images "absURL" "." }}
}
</script>
```

上面使用了`apply`函数，也暴露了Go模板解析器如何对`<script>`标签内的对象进行JSON编码。关于如何告诉Hugo不对此类标签内的字符串进行转义，请参阅[safeJS模板函数](https://gohugo.io/functions/safejs)的例子。

>   `absURL`和`relURL`对丢失的`/`很聪明，但如果不存在`/`，它们不会将其添加到URL中。

# `after`

在将一个数组切成仅在第N个项目之后的项目后。

语法：

```shell
after INDEX COLLECTION
```

以下是与`slice`函数结合使用`after`的情况。

```shell
{{ $data := slice "one" "two" "three" "four" }}
{{ range after 2 $data }}
    {{ . }}
{{ end }}
→ ["three", "four"]
```

## `after`与`first`的例子：第2-4个最近的文章 

你可以将`after`与`first`函数和Hugo强大的排序方法结合使用。让我们假设你在`example.com/articles`有一个列表页。你有10篇文章，但你希望你的列表/栏目页的模板只显示两行。

1.  最上面一行的标题是 "精选"，只显示最近发表的文章（即按内容文件中的`publishdate`）。
2.  第二行的标题是 "最近的文章"，只显示最近发表的第二至第四篇文章。

```html
<!-- layouts/section/articles.html -->

{{ define "main" }}
<section class="row featured-article">
    <h2>Featured Article</h2>
    {{ range first 1 .Pages.ByPublishDate.Reverse }}
     <header>
        <h3><a href="{{.Permalink}}">{{.Title}}</a></h3>
    </header>
    <p>{{.Description}}</p>
    {{ end }}
</section>
<div class="row recent-articles">
    <h2>Recent Articles</h2>
    {{ range first 3 (after 1 .Pages.ByPublishDate.Reverse) }}
        <section class="recent-article">
            <header>
                <h3><a href="{{.Permalink}}">{{.Title}}</a></h3>
            </header>
            <p>{{.Description}}</p>
        </section>
    {{ end }}
</div>
{{ end }}
```

# `anchorize`

接受一个字符串，并以与`defaultMarkdownHandler`对`markdown`头的处理相同的方式对其进行处理。

语法：

```shell
anchorize INPUT
```

如果`Goldmark`被设置为`defaultMarkdownHandler`，处理逻辑就会遵守`markup.goldmark.parser.autoHeadingIDType`的设置。

由于`defaultMarkdownHandler`和这个模板函数使用相同的处理逻辑，你可以使用后者来确定一个标题的ID，以便用锚标签进行链接。

```shell
{{ anchorize "This is a header" }} --> "this-is-a-header"
{{ anchorize "This is also    a header" }} --> "this-is-also----a-header"
{{ anchorize "main.go" }} --> "maingo"
{{ anchorize "Article 123" }} --> "article-123"
{{ anchorize "<- Let's try this, shall we?" }} --> "--lets-try-this-shall-we"
{{ anchorize "Hello, 世界" }} --> "hello-世界"
```

# `append`

append将一个或多个值追加到一个切片中，并返回结果切片。

语法：

```shell
COLLECTION | append VALUE [VALUE]...
```

```shell
COLLECTION | append COLLECTION
```

一个追加单个值的例子:

```shell
{{ $s := slice "a" "b" "c" }}
{{ $s = $s | append "d" "e" }}
{{/* $s now contains a []string with elements "a", "b", "c", "d", and "e" */}}
```

同样的例子，将一个切片追加到另一个切片。

```shell
{{ $s := slice "a" "b" "c" }}
{{ $s = $s | append (slice "d" "e") }}
```

`append`函数适用于所有类型，包括`Page`。

# `apply`

给定一个`map`、`array`或`slice`，`apply`返回一个新的`slice`并在其上应用一个函数。

语法：

```shell
apply COLLECTION FUNCTION [PARAM...]
```

`apply`期望至少有三个参数，这取决于被应用的函数。

1.  第一个参数是要操作的序列。
2.  第二个参数是字符串形式的函数名称，它必须是一个有效的Hugo函数的名称。
3.  然后，提供应用函数的参数，用字符串`". "`代表函数要应用的序列的每个元素。

下面是一个内容文件的例子，其中有`name:`作为前言字段。

```yaml
+++
names: [ "Derek Perkins", "Joe Bergevin", "Tanner Linsley" ]
+++
```

然后你可以按以下方式使用`apply`。

```shell
{{ apply .Params.names "urlize" "." }}
```

这将得到以下结果：

```shell
"derek-perkins", "joe-bergevin", "tanner-linsley"
```

这大致上相当于使用以下的`range`:

```shell
{{ range .Params.names }}{{ . | urlize }}{{ end }}
```

然而，不可能向`delimit`函数提供一个范围的输出，所以你需要`apply`它。

如果你有`post-tag-list.html`和`post-tag-link.html`作为`partials`，你可以分别使用以下片段:

```html
<!-- layouts/partials/post-tag-list.html -->

{{ with .Params.tags }}
<div class="tags-list">
  Tags:
  {{ $len := len . }}
  {{ if eq $len 1 }}
    {{ partial "post-tag-link.html" (index . 0) }}
  {{ else }}
    {{ $last := sub $len 1 }}
    {{ range first $last . }}
      {{ partial "post-tag-link.html" . }},
    {{ end }}
    {{ partial "post-tag-link.html" (index . $last) }}
  {{ end }}
</div>
{{ end }}
```

```html
<!-- layouts/partials/post-tag-link.html -->
<a class="post-tag post-tag-{{ . | urlize }}" href="/tags/{{ . | urlize }}">{{ . }}</a>
```

这样可行，但`post-tag-list.html`的复杂性相当高。Hugo模板需要对只有一个标签的情况进行特殊处理，而且它必须把最后一个标签作为特殊处理。此外，由于HTML的生成方式和浏览器的解释方式，标签列表将呈现为`Tags: tag1 , tag2 , tag3`这样的内容。

这个`layouts/partials/post-tag-list.html`的第一个版本将所有的操作分开，以方便阅读。接下来显示的是合并后的、更简化的版本。

```html
{{ with .Params.tags }}
    <div class="tags-list">
      Tags:
      {{ $sort := sort . }}
      {{ $links := apply $sort "partial" "post-tag-link.html" "." }}
      {{ $clean := apply $links "chomp" "." }}
      {{ delimit $clean ", " }}
    </div>
{{ end }}
```

现在在完成的版本中，你可以对标签进行排序，用`layouts/partials/post-tag-link.html`将标签转换为链接，去掉游离的换行符，并将标签连接到一个带分隔符的列表中以便展示。下面是前面的例子的一个简化的版本。

```html
<!-- layouts/partials/post-tag-list.html -->

{{ with .Params.tags }}
<div class="tags-list">
    Tags:
    {{ delimit (apply (apply (sort .) "partial" "post-tag-link.html" ".") "chomp" ".") ", " }}
</div>
{{ end }}
```

当通过一个管道接收序列作为参数时，`apply`不工作。

# `base64`

`base64Encode`和`base64Decode`让你轻松地用base64编码解码内容，反之则通过管道解码。

语法：

```shell
base64Decode INPUT
```

```shell
base64Encode INPUT
```

一个例子:

```html
<!-- base64-input.html -->

<p>Hello world = {{ "Hello world" | base64Encode }}</p>
<p>SGVsbG8gd29ybGQ = {{ "SGVsbG8gd29ybGQ=" | base64Decode }}</p>
```

```html
<!-- base-64-output.html -->

<p>Hello world = SGVsbG8gd29ybGQ=</p>
<p>SGVsbG8gd29ybGQ = Hello world</p>
```

你也可以把其他数据类型作为参数传给模板函数，模板函数会尝试转换它们。下面将把42从一个整数转换成一个字符串，因为`base64Encode`和`base64Decode`总是返回一个字符串。

```shell
{{ 42 | base64Encode | base64Decode }}
=> "42" rather than 42
```

## `base64`与API的关系 

如果我们必须处理来自API的响应，使用base64解码和编码就变得非常强大。

```shell
{{ $resp := getJSON "https://api.github.com/repos/gohugoio/hugo/readme"  }}
{{ $resp.content | base64Decode | markdownify }}
```

GitHub API 的响应包含 Hugo 仓库中 README.md 的 base64 编码版本。现在我们可以对其进行解码并解析Markdown。最终的输出结果将与 GitHub 上的渲染版本相似。

# `chomp`

删除任何尾部的换行字符。

语法：

```shell
chomp INPUT
```

在一个管道中，用于移除由其他处理（例如markdownify）添加的新行。

```shell
{{chomp "<p>Blockhead</p>\n"}} → "<p>Blockhead</p>"
```

# complement

`collections.Complement`（别名:`complement`,补集）给出了一个集合中不在任何其他集合中的元素。

语法：

```shell
COLLECTION | complement COLLECTION [COLLECTION]...
```

例子：

```shell
{{ $pages := site.RegularPages | first 50 }}
{{ $news := where $pages "Type" "news" | first 5 }}
{{ $blog := where $pages "Type" "blog" | first 5 }}
{{ $other := $pages | complement $news $blog | first 10 }}
```

以上是对主页的一个想象中的用例，你想在页面的不同地方用栏目/框显示不同的页面列表。5个来自`news`，5个来自`blog`，然后是其他列表中没有显示的10个页面，作为补集。

# `cond`

返回两个参数中的一个，取决于第三个参数的值。

类似三元运算符。

语法：

```shell
cond CONTROL VAR1 VAR2
```

`cond`如果CONTROL为真，则返回VAR1，如果不是，则返回VAR2。

例子：

```shell
{{ cond (eq (len $geese) 1) "goose" "geese" }}
```

如果`$geese`数组中正好有1个项目，将发出 "goose"，否则将发出 "geese"。

>   每当你使用`cond`函数时，两个变量表达式总是被评估。这意味着像 `cond false (div 1 0) 27` 这样的用法会产生一个错误，因为即使条件是假的，`div 1 0` 也会被评估。

>   换句话说，`cond`函数不提供短路求值，也不像普通的三元操作符那样，如果条件返回为`false`，就会把第一个表达式传递过去。

# `countrunes`

确定一个字符串中的字符数量，不包括任何空白符。

语法：

```shell
countrunes INPUT
```

`countwords`函数计算字符串中的每一个字，与此相反，`countrunes`函数确定内容中字符的数量，并排除任何空白字符。如果你处理的是类似中日韩的语言，这有特别的用处。

```shell
{{ "Hello, 世界" | countrunes }}
<!-- outputs a content length of 8 runes. -->
```

# `countwords`

计算一个字符串中的字数。

语法：

```shell
countwords INPUT
```

该模板功能的作用类似于`.WordCount`页面变量。

```shell
{{ "Hugo is a static site generator." | countwords }}
<!-- outputs a content length of 6 words.  -->
```

# `default`

允许设置一个默认值，如果没有设置第一个值，返回默认值。

语法：

```shell
default DEFAULT INPUT
```

`default`检查一个给定的值是否被设置，如果没有则返回一个默认值。在这种情况下，设置的含义根据数据类型的不同而不同。

-   数字类型和时间的非零值
-   对于字符串、数组、片断和地图来说，长度不为零
-   任何布尔值或结构值
-   任何其他类型的非零值

`default`函数的例子参考以下内容页。

```yaml
# content/posts/default-function-example.md
---
title: Sane Defaults
seo_title:
date: 2017-02-18
font:
oldparam: The default function helps make your templating DRYer.
newparam:
---
```

`default`，可以用多种方式来写。

```shell
{{ index .Params "font" | default "Roboto" }}
{{ default "Roboto" (index .Params "font") }}
```

上述两个`default`函数调用都返回`Roboto`。

然而，默认值不需要像前面的例子那样被硬编码。默认值可以是一个变量，也可以使用点符号直接从前面的内容中提取。

```html
<!-- variable-as-default-value.html -->

{{$old := .Params.oldparam }}
<p>{{ .Params.newparam | default $old }}</p>
```

这将返回:

```html
<p>The default function helps make your templating DRYer.</p>
```

然后使用点符号:

```html
<!-- dot-notation-default-value.html -->

<title>{{ .Params.seo_title | default .Title }}</title>
```

这将返回:

```html
<!-- dot-notation-default-return-value.html -->

<title>Sane Defaults</title>
```

下面的例子有相同的返回值，但远没有那么简练。这证明了默认值的效用。

使用`if`:

```html
<!-- if-instead-of-default.html -->

<title>{{if .Params.seo_title}}{{.Params.seo_title}}{{else}}{{.Title}}{{end}}</title>
=> Sane Defaults
```

使用`with`:

```html
<!-- with-instead-of-default.html -->

<title>{{with .Params.seo_title}}{{.}}{{else}}{{.Title}}{{end}}</title>
=> Sane Defaults
```

# `delimit`

遍历任何数组、切片或map，并返回一个由分隔符分隔的所有数值的字符串。

语法：

```shell
delimit COLLECTION DELIMIT LAST
```

在你的模板中调用`delimit`的形式为

```shell
{{ delimit array/slice/map delimiter optionallastdelimiter}}
```

`delimit `遍历任何数组、切片或map，并返回一个由分隔符（即函数调用的第二个参数）分隔的所有值的字符串。有一个可选的第三个参数，让你选择一个不同的定界符，放在循环的最后两个值之间。



为了保持一致的输出顺序，map将按照键进行排序，并且只返回值的部分。

后面的`delimit`的例子都使用了相同的前述内容。

```toml
# delimit-example-front-matter.toml
+++
title: I love Delimit
tags: [ "tag1", "tag2", "tag3" ]
+++
```

```html
<!-- delimit-page-tags-input.html -->

<p>Tags: {{ delimit .Params.tags ", " }}</p>
```

```html
<!-- delimit-page-tags-output.html -->

<p>Tags: tag1, tag2, tag3</p>
```

下面是同一个例子，但有可选的 "最后 "分隔符。

```html
<!-- delimit-page-tags-final-and-input.html -->

Tags: {{ delimit .Params.tags ", " ", and " }}
```

```html
<!-- delimit-page-tags-final-and-output.html -->

<p>Tags: tag1, tag2, and tag3</p>
```

# `dict`

从一个键和值对的列表中创建一个字典。

语法：

```shell
dict KEY VALUE [KEY VALUE]...
```

`dict`对于向部分模板传递一个以上的值特别有用。

注意，`key`可以是一个`string`，也可以是一个`string slice`	。后者对于创建一个深度嵌套的结构很有用，例如:

```shell
{{ $m := dict (slice "a" "b" "c") "value" }}
```

### 例子: 使用`dict`将多个值传递给一个`partial` 

下面的`partial` 创建了一个SVG，并期望从调用者那里获得填充、高度和宽度。

#### `partial` 定义

```svg
<!-- layouts/partials/svgs/external-links.svg -->

<svg version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
fill="{{ .fill }}" width="{{ .width }}" height="{{ .height }}" viewBox="0 0 32 32" aria-label="External Link">
<path d="M25.152 16.576v5.696q0 2.144-1.504 3.648t-3.648 1.504h-14.848q-2.144 0-3.648-1.504t-1.504-3.648v-14.848q0-2.112 1.504-3.616t3.648-1.536h12.576q0.224 0 0.384 0.16t0.16 0.416v1.152q0 0.256-0.16 0.416t-0.384 0.16h-12.576q-1.184 0-2.016 0.832t-0.864 2.016v14.848q0 1.184 0.864 2.016t2.016 0.864h14.848q1.184 0 2.016-0.864t0.832-2.016v-5.696q0-0.256 0.16-0.416t0.416-0.16h1.152q0.256 0 0.416 0.16t0.16 0.416zM32 1.152v9.12q0 0.48-0.352 0.8t-0.8 0.352-0.8-0.352l-3.136-3.136-11.648 11.648q-0.16 0.192-0.416 0.192t-0.384-0.192l-2.048-2.048q-0.192-0.16-0.192-0.384t0.192-0.416l11.648-11.648-3.136-3.136q-0.352-0.352-0.352-0.8t0.352-0.8 0.8-0.352h9.12q0.48 0 0.8 0.352t0.352 0.8z"></path>
</svg>
```

`partial` 调用

填充、高度和宽度的值可以用`dict`存储在一个对象中，并传递给`partial` 。

```html
<!-- layouts/_default/list.html -->
{{ partial "svgs/external-links.svg" (dict "fill" "#01589B" "width" 10 "height" 20 ) }}
```

# `echoParam`

如果一个参数被设置，则打印该参数。

语法：

```shell
echoParam DICTIONARY KEY
```

```shell
{{ echoParam .Params "project_url" }}
```

# `emojify`

通过Emoji表情符号处理器运行一个字符串。

语法：

```shell
emojify INPUT
```

`emoji`通过Emoji表情符号处理器运行一个传递的字符串。

关于可用的表情符号，请参见[Emoji速查表](https://www.webfx.com/tools/emoji-cheat-sheet/)。

`emojify`函数可以在你的模板中调用，但默认情况下不能直接在你的内容文件中调用。对于内容文件中的表情符号，在你的网站配置中设置`enableEmoji`为`true`。然后你就可以直接在你的内容文件中写入表情符号速记；例如:`I :heart: Hugo!`。

I ❤️ Hugo!

# `eq`

返回arg1 == arg2的布尔值。

语法：

```shell
eq ARG1 ARG2
```

```shell
{{ if eq .Section "blog" }}current{{ end }}
```

# `errorf and warnf`

从模板中记录ERROR或WARNING。

语法：

```shell
errorf FORMAT INPUT
```

`errorf`或`warnf`将评估一个格式化的字符串，然后将结果输出到ERROR或WARNING日志（每条错误信息只输出一次，以避免淹没日志）。

任何ERROR也会导致构建失败（hugo命令将退出`exit -1`）。

这两个函数都返回一个空字符串，所以信息只被打印到控制台。

```shell
{{ errorf "Failed to handle page %q" .Path }}
```

```shell
{{ warnf "You should update the shortcodes in %q" .Path }}
```

注意，`errorf`、`erroridf`和`warnf`支持`fmt`包的所有格式化动词。

## 抑制错误 

有时，让用户压制一个错误并使构建成功可能是有意义的。

你可以通过使用`erroridf`函数来做到这一点。这个函数需要一个错误ID作为第一个参数。

```shell
{{ erroridf "my-custom-error" "You should consider fixing this." }}
```

这将输出;

```shell
ERROR 2021/06/07 17:47:38 You should consider fixing this.
If you feel that this should not be logged as an ERROR, you can ignore it by adding this to your site config:
ignoreErrors = ["my-custom-error"]
```

# `fileExists`

检查文件或目录是否存在。

语法：

```shell
os.FileExists PATH
```

```shell
fileExists PATH
```

`os.FileExists`函数试图解析相对于你的项目目录根的路径。如果没有找到匹配的文件或目录，它将试图解析相对于`contentDir`的路径。前面的路径分隔符（`/`）是可选的。

有了这个目录结构:

```shell
content/
├── about.md
├── contact.md
└── news/
    ├── article-1.md
    └── article-2.md
```

该函数返回这些值。

```shell
{{ os.FileExists "content" }} --> true
{{ os.FileExists "content/news" }} --> true
{{ os.FileExists "content/news/article-1" }} --> false
{{ os.FileExists "content/news/article-1.md" }} --> true
{{ os.FileExists "news" }} --> true
{{ os.FileExists "news/article-1" }} --> false
{{ os.FileExists "news/article-1.md" }} --> true
```

# `findRE`

返回一个符合正则表达式的字符串列表。

语法：

```shell
findRE PATTERN INPUT [LIMIT]
```

默认情况下，所有的匹配将被包括在内。匹配的数量可以用一个可选的第三个参数来限制。

下面的例子返回内容中所有二级标题（`<h2>`）的列表。

```shell
{{ findRE "<h2.*?>(.|\n)*?</h2>" .Content }}
```

你可以用第三个参数来限制列表中的匹配数量。下面的例子显示了如何将返回值限制为只有一个匹配项（或者没有，如果没有匹配的子字符串）。

```shell
{{ findRE "<h2.*?>(.|\n)*?</h2>" .Content 1 }}
    <!-- returns ["<h2 id="#foo">Foo</h2>"] -->
```

>   Hugo使用Go的正则表达式包，它与Perl、Python和其他语言使用的一般语法相同，但对于那些来自PCRE背景的人来说有一些小的区别。关于完整的语法列表，请看[GitHub的re2维基](https://github.com/google/re2/wiki/Syntax)。

>   如果你刚开始学习RegEx，或者至少是Go的风味，你可以在浏览器中练习模式匹配，网址是https://regex101.com/。

# `first`

将一个数组切成只有前N个元素。

语法：

```shell
first LIMIT COLLECTION
```

`first`的工作方式类似于SQL中的`limit`关键字。它将数组减少到只有前N个元素。它把数组和元素的数量作为输入。

`first`需要两个参数:

1.  `number of elements`
2.  `array` *or* `slice of maps or structs`

```shell
# layout/_default/section.html

{{ range first 10 .Pages }}
    {{ .Render "summary" }}
{{ end }}
```

注意：与第一项无关，`LIMIT`可以为'0'，以返回一个空数组。

## 使用`first`和`where`

同时使用`first`和`where`可以非常强大。下面的片段只从主要部分获得一个帖子列表，按标题参数排序，然后只对该列表中的前5个帖子进行排序。

```shell
# first-and-where-together.html
{{ range first 5 (where site.RegularPages "Type" "in" site.Params.mainSections).ByTitle }}
   {{ .Content }}
{{ end }}
```

# `float`

从传入函数的参数中创建一个浮点数。

语法：

```shell
float INPUT
```

有助于将字符串变成浮点数。

```shell
{{ float "1.23" }} → 1.23
```

# `ge`

返回arg1 >= arg2的布尔值。

语法：

```shell
ge ARG1 ARG2
```

```shell
{{ if ge 10 5 }}true{{ end }}
```

# `getenv`

返回一个环境变量的值，如果环境变量没有被设置，则返回一个空字符串。

语法：

```shell
os.Getenv VARIABLE
```

```shell
getenv VARIABLE
```

例子：

```shell
{{ os.Getenv "HOME" }} --> /home/victor
{{ os.Getenv "USER" }} --> victor
```

你可以在建立你的网站时传递环境变量：

```shell
MY_VAR1=foo MY_VAR2=bar hugo

OR

export MY_VAR1=foo
export MY_VAR2=bar
hugo
```

然后在一个模板内检索这些值:

```shell
{{ os.Getenv "MY_VAR1" }} --> foo
{{ os.Getenv "MY_VAR2" }} --> bar
```

# `group`

`group`将一个页面列表分组。

语法：

```shell
PAGES | group KEY
```

```html
<!-- layouts/partials/groups.html -->

{{ $new := .Site.RegularPages | first 10 | group "New" }}
{{ $old := .Site.RegularPages | last 10 | group "Old" }}
{{ $groups := slice $new $old }}
{{ range $groups }}
<h3>{{ .Key }}{{/* Prints "New", "Old" */}}</h3>
<ul>
    {{ range .Pages }}
    <li>
    <a href="{{ .Permalink }}">{{ .Title }}</a>
    <div class="meta">{{ .Date.Format "Mon, Jan 2, 2006" }}</div>
    </li>
    {{ end }}
</ul>
{{ end }}
```

你从`group`得到的页面组与你从Hugo中内置的`group`方法得到的类型相同。上面的例子甚至可以进行分页。

# `gt`

返回arg1 > arg2的布尔值。

语法：

```shell
gt ARG1 ARG2
```

```shell
{{ if gt 10 5 }}true{{ end }}
```

# `hasPrefix`

测试一个字符串是否以前缀开始。

语法：

```shell
hasPrefix STRING PREFIX
```

```shell
{{ hasPrefix "Hugo" "Hu" }} → true
```

# `highlight`

用语法高亮器渲染代码。

语法：

```shell
transform.Highlight INPUT LANG [OPTIONS]
```

```shell
highlight INPUT LANG [OPTIONS]
```

高亮功能使用Chroma语法高亮器，支持200多种语言，有40多种可用样式。

## Parameters

**INPUT**
要突出显示的代码。

**LANG**

要突出显示的代码的语言。从支持的语言中选择一个。不区分大小写。

**OPTIONS**

一个可选的、用逗号隔开的零或多个选项的列表。在站点配置中设置默认值。

## Options

**lineNos**

Boolean. Default is `false`.
在每一行的开头显示一个数字。

**lineNumbersInTable**

Boolean. Default is `true`.
在一个有两个单元格的HTML表格中显示高亮显示的代码。左边的表格单元包含行号。右边的表格单元包含代码，允许用户选择和复制没有行号的代码。如果`lineNos`为`false`，则无关紧要。

**anchorLineNos**

Boolean. Default is `false`.

将每个行号渲染成一个HTML锚点元素，并将周围`<span>`的`id`属性设置为行号。如果`lineNos`为`false`，则无关紧要。

**lineAnchors**

String. Default is `""`.

当把行号渲染成HTML锚点元素时，把这个值前置到周围`<span>`的`id`属性上。当一个页面包含两个或多个代码块时，这提供了唯一的id属性。如果`lineNos`或`anchorLineNos`为`false`，则无关紧要。

**lineNoStart**

Integer. Default is `1`.
显示在第一行开头的数字。如果`lineNos`为`false`，则无关紧要。

**hl_Lines**

String. Default is `""`.
以空格分隔的要强调的代码行的列表。要强调第2、3、4和7行，将此值设置为`2-4 7`。这个选项与`lineNoStart`选项无关。

**style**

String. Default is `monokai`.

应用于突出显示的代码的CSS样式。参见[样式库](https://xyproto.github.io/splash/docs/)的例子。区分大小写。

**noClasses**

Boolean. Default is `true`.

使用内联CSS样式而不是外部CSS文件。要使用外部CSS文件，请将此值设置为`false`，并使用[hugo客户端生成文件](https://gohugo.io/commands/hugo_gen_chromastyles/)。

**tabWidth**

Integer. Default is `4`.

在你突出显示的代码中，用这个数量的空格代替每个制表符。

**guessSyntax**

Boolean. Default is `false`.

如果`LANG`参数是空的或者是一个不被识别的语言，如果可能的话，自动检测语言，否则就使用后备语言。

>   你可以使用以下速记符号，而不是同时指定`lineNos`和`lineNumbersInTable`
>
>   `lineNos=inline`
>   相当于`lineNos=true`，`lineNumbersInTable=false`
>   `lineNos=table`
>   相当于`lineNos=true` 和 `lineNumbersInTable=true`

**例子**:

```shell
{{ $input := `fmt.Println("Hello World!")` }}
{{ transform.Highlight $input "go" }}

{{ $input := `console.log('Hello World!');` }}
{{ $lang := "js" }}
{{ transform.Highlight $input $lang "lineNos=table, style=api" }}

{{ $input := `echo "Hello World!"` }}
{{ $lang := "bash" }}
{{ $options := slice "lineNos=table" "style=dracula" }}
{{ transform.Highlight $input $lang (delimit $options ",") }}
```

# `hmac`

返回一个使用密钥签署信息的加密哈希值。

语法:

```shell
crypto.HMAC HASH_TYPE KEY MESSAGE [ENCODING]
```

```shell
hmac HASH_TYPE KEY MESSAGE [ENCODING]
```

将`HASH_TYPE`参数设置为`md5`, `sha1`, `sha256`, 或 `sha512`。

将可选的`ENCODING`参数设置为十六进制（默认）或二进制。

```shell
{{ hmac "sha256" "Secret key" "Secret message" }}
5cceb491f45f8b154e20f3b0a30ed3a6ff3027d373f85c78ffe8983180b03c84

{{ hmac "sha256" "Secret key" "Secret message" "hex" }}
5cceb491f45f8b154e20f3b0a30ed3a6ff3027d373f85c78ffe8983180b03c84

{{ hmac "sha256" "Secret key" "Secret message" "binary" | base64Encode }}
XM60kfRfixVOIPOwow7Tpv8wJ9Nz+Fx4/+iYMYCwPIQ=
```

# `htmlEscape`

返回给定的字符串，并转义保留的HTML代码。

语法:

```shell
htmlEscape INPUT
```

在结果中`&`成为`&amp;`等等。它只转义`<`, `>,` `&`, `' `和` "`.

```shell
{{ htmlEscape "Hugo & Caddy > WordPress & Apache" }} → "Hugo &amp; Caddy &gt; WordPress &amp; Apache"
```

# `htmlUnescape`

返回给定的HTML转义代码未转义的字符串。

语法:

```shell
htmlUnescape INPUT
```

`htmlUnescape` 返回给定的字符串，其中的HTML转义代码没有被转义。

如果需要完全未转义的字符，请记住将此输出传递给 `safeHTML`。否则，输出将被再次转义为正常的字符。

```shell
{{ htmlUnescape "Hugo &amp; Caddy &gt; WordPress &amp; Apache" }} → "Hugo & Cadd
```

# `hugo`

`hugo`函数提供了对Hugo相关数据的便捷访问。

语法:

```shell
hugo
```

`hugo`返回一个包含以下函数的实例。

**hugo.Generator**

`hugo.Generator`输出一个完整的元数据HTML标签；例如，`<meta name="generator" content="Hugo 0.99.1" />`

**hugo.Version**

你正在使用的Hugo二进制文件的当前版本，例如`0.99.1`。

**hugo.Environment**

通过`--environment` cli标签定义的当前运行环境。

**hugo.CommitHash**

当前Hugo二进制文件的git提交哈希值，例如`0e8bed9ccffba0df554728b46c5bbf6d78ae5247`。

**hugo.BuildDate**

当前Hugo二进制文件的编译日期，格式为RFC 3339，例如：`2002-10-02T10:00:00-05:00`

**hugo.IsExtended**

这是否是扩展的Hugo二进制。

**hugo.IsProduction**

如果`hugo.Environment`被设置为生产环境，则返回`true`。

>   我们强烈建议在您网站的`<head>`中使用`hugo.Generator`。`hugo.Generator`默认包含在`themes.gohugo.io`上托管的所有主题中。发电机标签允许Hugo团队跟踪Hugo的使用情况和受欢迎程度。

## `hugo.Deps`

`hugo.Deps`返回一个项目的依赖性列表（Hugo模块或本地主题组件）。

每个依赖项都包含:

### **Path (string)**

返回该模块的路径。这将是模块的路径，例如 `"github.com/gohugoio/myshortcodes"`，或你的`/theme`文件夹下面的路径，例如` "mytheme"`。

**Version (string)**

该模块的版本。

**Vendor (bool)**

这种依赖关系是否被出售。

**Time (time.Time)**

创建的时间版本。

**Owner**

在依赖关系树中，这是第一个将该模块定义为依赖关系的模块。

**Replace (\*Dependency)**

被这个依赖性所取代。

一个列出依赖关系的例子:

```html
 <h2>Dependencies</h2>
<table class="table table-dark">
  <thead>
    <tr>
      <th scope="col">#</th>
      <th scope="col">Owner</th>
      <th scope="col">Path</th>
      <th scope="col">Version</th>
      <th scope="col">Time</th>
      <th scope="col">Vendor</th>
    </tr>
  </thead>
  <tbody>
    {{ range $index, $element := hugo.Deps }}
    <tr>
      <th scope="row">{{ add $index 1 }}</th>
      <td>{{ with $element.Owner }}{{.Path }}{{ end }}</td>
      <td>
        {{ $element.Path }}
        {{ with $element.Replace}}
        => {{ .Path }}
        {{ end }}
      </td>
      <td>{{ $element.Version }}</td>
      <td>{{ with $element.Time }}{{ . }}{{ end }}</td>
      <td>{{ $element.Vendor }}</td>
    </tr>
    {{ end }}
  </tbody>
</table>
```

# `humanize`

返回一个参数的人性化版本，第一个字母大写。

语法：

```shell
humanize INPUT
```

如果输入的是一个int64值或者是一个整数的字符串表示，humanize会返回附加了适当序号的数字。

```shell
{{humanize "my-first-post"}} → "My first post"
{{humanize "myCamelPost"}} → "My camel post"
{{humanize "52"}} → "52nd"
{{humanize 103}} → "103rd"
```

# `i18n`

根据你的i18n配置文件来翻译一段内容。

语法：

```shell
i18n KEY
```

```shell
T KEY
```

这是根据你的`i18n/en-US.toml`文件来翻译一段内容。你可以使用`go-i18n`工具来管理你的翻译。翻译可以存在于主题和版本库的根目录中。

```shell
{{ i18n "translation_id" }}
```

>   `T`是`i18n`的一个别名。例如，`{{ T "translation_id" }}`。

### Query a flexible translation with variables

通常你会想在翻译字符串中使用页面变量。要做到这一点，在调用`i18n`时要传递`.`context。

```shell
{{ i18n "wordCount" . }}
```

该函数将把`.`context传递给` "wordCount "`id:

```yaml
# i18n/en-US.yaml

wordCount:
  other: This article has {{ .WordCount }} words.
```

假设`.WordCount`在上下文中的值是101。其结果将是:

```shell
This article has 101 words.
```

有关字符串翻译的更多信息，请参阅[《多语言模式下的字符串翻译》](https://gohugo.io/content-management/multilingual/#translation-of-strings)。

# `Image Filters`

`images`命名空间提供了一个过滤器和其他图像相关功能的列表。
参见[images.Filter](https://gohugo.io/functions/images/#filter)，了解如何将这些过滤器应用到图像上。

## Overlay

[New in v0.80.0](https://github.com/gohugoio/hugo/releases/tag/v0.80.0)

叠加创建一个过滤器，在x,y位置叠加源图像，例如:

```shell
{{ $logoFilter := (images.Overlay $logo 50 50 ) }}
{{ $img := $img | images.Filter $logoFilter }}
```

如果你只需要应用一次过滤器，则这是上述的一个简短版本:

```shell
{{ $img := $img.Filter (images.Overlay $logo 50 50 )}}
```

以上将在`$img`的左上角覆盖`$logo`（在`x=50`，`y=50`的位置）。

## Text 

[New in v0.90.0](https://github.com/gohugoio/hugo/releases/tag/v0.90.0)

使用`Text`过滤器，你可以在图像上添加文本。

下面的例子将以指定的颜色、大小和位置将文字`Hugo rocks！`添加到图片上。

```shell
{{ $img := resources.Get "/images/background.png"}}
{{ $img = $img.Filter (images.Text "Hugo rocks!" (dict
    "color" "#ffffff"
    "size" 60
    "linespacing" 2
    "x" 10
    "y" 20
))}}
```

如果需要，你可以加载一个自定义字体。将字体作为Hugo资源加载，并将其设置为一个选项。

```shell
{{ $font := resources.Get "https://github.com/google/fonts/raw/main/apache/roboto/static/Roboto-Black.ttf" }}
{{ $img := resources.Get "/images/background.png"}}
{{ $img = $img.Filter (images.Text "Hugo rocks!" (dict
    "font" $font
))}}
```

## **Brightness**

`Brightness`创建一个过滤器，改变图像的亮度。百分比参数必须在（-100，100）范围内。

## ColorBalance

`ColorBalance`创建一个过滤器，改变图像的色彩平衡。每个颜色通道（红、绿、蓝）的百分比参数必须在（-100, 500）范围内。

## Colorize

`Colorize`创建一个过滤器，产生一个图像的彩色版本。色调参数是色轮上的角度，通常在（0，360）范围内。饱和度参数必须在（0，100）范围内。百分比参数指定效果的强度，它必须在（0，100）范围内。

## Contrast

`Contrast`创建一个过滤器，改变图像的对比度。百分比参数必须在（-100，100）范围内。

## Gamma

`Gamma`创建一个过滤器，对图像进行伽玛校正。伽马参数必须是正数。伽玛=1给出原始图像。小于1的伽玛会使图像变暗，大于1的伽玛会使图像变亮。

## GaussianBlur

高斯模糊（GaussianBlur）创建一个过滤器，对图像进行高斯模糊处理。

## Grayscale

`Grayscale`创建一个过滤器，产生一个图像的灰度版本。

## Hue

`Hue`创建一个过滤器，旋转图像的色调。色调角的移动范围通常为-180到180。

## Invert

`Invert`创建一个过滤器，翻转了图像的颜色。

## Pixelate

`Pixelate`创建了一个过滤器，将像素化的效果应用到图像上。

## Saturation

`Saturation`创建一个过滤器，改变图像的饱和度。

## Sepia

`Sepia`创建一个过滤器，产生一个棕褐色的图像版本。

## Sigmoid

`Sigmoid`创建了一个过滤器，它使用一个`sigmoidal`函数改变图像的对比度，并返回调整后的图像。这是一个非线性的对比度变化，对照片调整很有用，因为它保留了高光和阴影的细节。

## UnsharpMask

`UnsharpMask`创建一个滤镜来锐化图像。`sigma`参数用于高斯函数，影响效果的半径。Sigma必须是正数。锐化半径大致等于`3*sigma`。数量参数控制边缘边界变得多深和多浅。通常是在0.5和1.5之间。阈值参数控制将被锐化的最小亮度变化。通常在0到0.05之间。

## Other Functions

### Filter

可用于对图像应用一组过滤器:

```shell
{{ $img := $img | images.Filter (images.GaussianBlur 6) (images.Pixelate 8) }}
```

### ImageConfig

解析图像并返回高度、宽度和颜色模型。

`imageConfig`函数接受一个参数，即相对于项目根目录的文件路径（字符串），可以有或没有前导斜线。

```shell
{{ with (imageConfig "favicon.ico") }}
favicon.ico: {{.Width}} x {{.Height}}
{{ end }}
```

# `in`

检查一个元素是否在一个数组或切片中，或在一个字符串中的子串中，并返回一个布尔值。

语法：

```shell
in SET ITEM
```

支持的元素有字符串、整数和浮点数，不过只有float64会如期匹配。

此外，`in`还可以检查一个字符串中是否存在子串。

```shell
{{ if in .Params.tags "Git" }}Follow me on GitHub!{{ end }}
```

```shell
{{ if in "this string contains a substring" "substring" }}Substring found!{{ end }}
```

# `index`

查找传递给它的数据结构的索引或键。

语法：

```shell
index COLLECTION INDEXES
```

```shell
index COLLECTION KEYS
```

`index`函数返回其第一个参数被以下参数索引的结果。每个被索引的项目必须是一个地图或一个切片，例如:

```shell
{{ $slice := slice "a" "b" "c" }}
{{ index $slice 1 }} => b
{{ $map := dict "a" 100 "b" 200 }}
{{ index $map "b" }} => 200
```

该函数以多个索引为参数，这可以用来获得嵌套的值，例如:

```shell
{{ $map := dict "a" 100 "b" 200 "c" (slice 10 20 30) }}
{{ index $map "c" 1 }} => 20
{{ $map := dict "a" 100 "b" 200 "c" (dict "d" 10 "e" 20) }}
{{ index $map "c" "e" }} => 20
```

你可以把多个索引写成一个切片:

```shell
{{ $map := dict "a" 100 "b" 200 "c" (dict "d" 10 "e" 20) }}
{{ $slice := slice "c" "e" }}
{{ index $map $slice }} => 20
```

## 例子：Load Data from a Path Based on Front Matter Params

假设你想在`content/vacations/`中写的每一篇文章的前言添加一个`location = ""` 字段。你想用这个字段在你的`single.html`模板中的文章底部填入有关位置的信息。你在`data/locations/`中也有一个目录，看起来像下面这样。

```shell
.
└── data
    └── locations
        ├── abilene.toml
        ├── chicago.toml
        ├── oslo.toml
        └── provo.toml
```

下面是一个例:

```shell
# data/locations/oslo.yaml
 
pop_city: 658390
pop_metro: 1717900
website: https://www.oslo.kommune.no
```

我们将使用的例子是一篇关于奥斯陆的文章，其前言应设置为与`data/locations/`中的相应文件名完全相同的名称。

```shell
title = "My Norwegian Vacation"
location = "oslo"
```

`oslo.toml`的内容可以通过以下节点路径从你的模板中访问：`.Site.Data.lots.oslo`。然而，你所需要的具体文件将根据前言的内容而改变。

这就是需要`index`函数的地方。`index`在这个用例中需要2个参数。

 	1. 节点路径
 	2. 一个对应于所需数据的字符串；例如:

```shell
{{ index .Site.Data.locations “oslo” }}
```



`.Params.location`的变量是一个字符串，因此可以取代上面例子中的`oslo`。

```shell
{{ index .Site.Data.locations .Params.location }}
=> map[website:https://www.oslo.kommune.no pop_city:658390 pop_metro:1717900]
```

现在，该调用将根据内容前言中指定的位置返回特定的文件，但你很可能想把特定的属性写到模板中。你可以通过点符号`.`沿着节点路径继续往下走来做到这一点。

```shell
{{ (index .Site.Data.locations .Params.location).pop_city }}
=> 658390
```

# `int`

从传入函数的参数中创建一个`int`。

语法：

```shell
int INPUT
```

有助于将字符串变成数字。

```shell
{{ int "123" }} → 123
```

>   如果输入的字符串应该代表一个十进制的数字，如果它有前面的0，那么在把字符串传递给`int`函数之前，必须把这些0去掉，否则这个字符串将被尝试解析为一个八进制的数字表示。
>
>   `strings.TrimLeft`函数可用于此目的。
>
>   ```shell
>   {{ int ("0987" | strings.TrimLeft "0") }}
>   {{ int ("00987" | strings.TrimLeft "0") }}
>   ```
>
>   **解释**
>
>   `int`函数最终调用了Go库`strconv`中的`ParseInt`函数。
>
>   从它的文档来看。
>
>   >   基数是由字符串的前缀暗示的：基数16代表 "0x"，基数8代表 "0"，否则就是基数10。

# `intersect`

返回两个数组或切片的共同元素，顺序与第一个数组相同。

语法：

```shell
intersect SET1 SET2
```

一个有用的例子是，当与`where`结合时，将其作为`AND`过滤器使用。

## AND filter in where query

```shell
{{ $pages := where .Site.RegularPages "Type" "not in" (slice "page" "about") }}
{{ $pages := $pages | union (where .Site.RegularPages "Params.pinned" true) }}
{{ $pages := $pages | intersect (where .Site.RegularPages "Params.images" "!=" nil) }}
```

以上是对非`page`或`about`类型的普通页面的检索，除非它们被钉住。最后，我们排除了所有在页面参数中没有设置`images`的页面。

# `isset`

如果该参数被设置，则返回真。

语法：

```shell
isset COLLECTION INDEX
```

```shell
isset COLLECTION KEY
```

接受一个切片、数组或通道和一个索引或一个map和一个键作为输入。

```shell
{{ if isset .Params "project_url" }} {{ index .Params "project_url" }}{{ end }}
```

>   所有站点级的配置键都以小写形式存储。因此，网站配置文件中的`myParam`键值需要用`{{if isset .Site.Params "myparam"}}`访问，而不是`{{if isset .Site.Params "myParam"}}`。注意，你仍然可以用`.Site.Params.myParam`或`.Site.Params.myparam`访问同一个配置键，例如，在使用`with`时。这一限制也适用于从快捷键中访问页面级前言的键。

# `jsonify`

将一个给定的对象编码为JSON。

语法：

```shell
jsonify INPUT
```

```shell
jsonify OPTIONS INPUT
```

Jsonify将一个给定的对象编码为JSON。

要自定义JSON的打印方式，可以在第一个参数中传递一个选项字典。支持的选项有 "prefix "和 "indent"。输出中的每个JSON元素将在一个新的行中开始，以prefix开始，后面是根据缩进嵌套的一个或多个副本的缩进。

```shell
{{ dict "title" .Title "content" .Plain | jsonify }}
{{ dict "title" .Title "content" .Plain | jsonify (dict "indent" "  ") }}
{{ dict "title" .Title "content" .Plain | jsonify (dict "prefix" " " "indent" "  ") }}
```

## Jsonify options

**indent ("")**

要使用的缩进。

**prefix ("")**

缩进的前缀。

**noHTMLEscape (false)**

禁止在JSON引号字符串中转义有问题的HTML字符。默认行为是将`&`、`<`和`>`转义为`\u0026`、`\u003c`和`\u003e`，以避免在HTML中嵌入JSON时可能出现的某些安全问题。

也请参见`.PlainWords`、`.Plain`和`.RawContent`页面变量。
