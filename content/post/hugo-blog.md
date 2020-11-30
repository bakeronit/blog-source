---
title:       "用Hugo构建博客"
date:        2020-11-29
author:      "ZJ"
image:       ""
tags:        ["web"]
categories:  ["Note" ]
notoc: true
---

之前学着用hexo和github page建了一个博客，虽然购买了几年的域名，但是几乎处于荒废状态。也没有研究具体的用法，最近周围的人很多都开始用jekyll来做一些静态的网页用来当自己的简历或者写教程，所以我也有了重新开始折腾博客的想法。最终我选择了用Hugo，只是因为它很快。和Hexo还有Jekyll一样，Hugo也是一个用于生成静态网页的框架。不同的是Hugo使用Go语言写的，Hexo和Jekyll分别用的是Node.js和Ruby。我选择了这个[CleanWhite]()的主题，和之前我hexo的主题很像。你也可以不用主题，直接在layout文件夹里定义自己的模板。

跟着网上的教程安装并配置好主题后，我简单的研究了一下hugo的结构和用法，尝试去理解一些技术层面的东西，但是并不深入。

## Content
所有的文章都在content文件夹下以markdown格式存在，使用 `hugo new filename.md`可以直接在content下生成一个空白的模板。可以将文件生成在content下的子文件夹，不存的文件夹在hugo会生成。产生的页面则是在对应的文件夹名下。例如，`hugo new dir1/filename.md`的对应页面是`domainname.com/dir1/filename`。

## Front Matter
Front Matter是markdown文件最上方用来预先定义参数的内容，下面是YAML的格式，TOML格式的语法非常相似，但是使用加号分隔，并且用等号赋值。Hugo支持YAML,TOML和JSON格式，Jekyll只支持YAML。
```YAML
---
 title: "title"
 date:  2020-11-28
 author: "anyone"
 tags: ["tag1"]
---
```
Front Matter 提前指定了一些变量(predefined var)，在html中调用, 如 <title>{{ .Title }}</title>。Hugo提供了很多预定义的变量，如title，date，image，categories，layout等。你可以有自己在front matter里定义自己的变量，Hugo会全部放到一个Param变量，然后通过{{ .Param.var }}来访问。你可以用变量weight来控制文章在listing页面的排序，想要置顶就是`weight:1`.

## Archetypes

Archetypes是创建新的post文章的模板文件,也是md文件，就在archtypes文件夹下面。
```markdown
---
title:       "{{ replace .TranslationBaseName "-" "-" | title }}"
date:        "{{ .Date }}"
draft:       true
author:      "JZ"
tags:        ["tag1", "tag2"]
categories:  ["Note" ]
---

** insert things here **

```
 可以定义多个archtypes作为不同类型文章的模板，例如有一个post.md。你可以通过`hugo new post/my-post.md`创建这个文件。

## Taxonomies
这个是一个用于对文章进行分组的功能，让用户可以根据文章的一些属性进行分类，也方便查询和阅读。Hugo默认的Taxonomies有tags和categories，都可以在front matter里直接定义，hugo会为每个标签或者category生成单独的页面。具体的用法取决于模板定义，我用的这个模板，会将不同的categories的文章放到一个单独的列表页面，在header里可以进行访问。

当然你也可以添加自己的taxonomies，比如说你想根据当天的天气，心情来分类，可以在front matter里定义：
```YAML
---
title:       "{{ replace .TranslationBaseName "-" "-" | title }}"
date:        "{{ .Date }}"
tags:        ["travel", "photo"]
categories:  ["Life" ]
moods:       ['happy']
weathers:    ['sunny']
---
```
但是因为不会默认的taxonomies，hugo不会为这个分类单独生成一个页面，所以需要在配置文件`config.toml`里设置。
```TOML
[taxonomies]
  tag = "tags"
  category = "categories"
  mood = "moods"
  weather = "weathers"
```
## Template
我用的是别人写好的主题，但是其实主题也是别人写好的模板。在hugo里有两种模板，一种是列表模板，一个是single模板。在`layout/_default`文件夹下的`list.html`和`single.html`就是列表页面和single页面的模板。列表页面用于显示多个文章，single页面就是显示一篇文章。这些页面都共享一些元素，比如说header，footer等。每一篇文章的页面基本结构都是一样的，除了文章本身的内容。所以用到模板就可以只建好一个html页面，然后用markdown解析的内容替换模板中的一些变量。

还有一些页面你可以自己定义，主题里的layout会被根目录里的模板覆盖。主页其实也是一个列表页面，如果你不想你的主页是一个简单的列表页面，或者你想在模板上进行改进。你可以在`layout/index.html`再创建一个home page模板。

如果你想对在content文件夹下的文章用单独的模板，可以使用section template。只需要在layout下单独添加`single.html`或者`list.html`。简单的说就是content里文件夹里的md文件会调用layout文件夹下对应文件夹名里的模板html。另外，因为博客网页的结构非常单一，所以还可以用一个base template来减少代码量。在`layout/_default`下的文件`baseof.html`就是整个网页的基本框架的模板。在`baseof.html`里，可以用到一个特殊的hugo实体block, 在html里可以插入一些block来占位，然后在另一个html里定义这个block的内容。

还有一种模板的用法是partial templates，是一种可以模块化的方法。可以把一个页面的元素拆分开来，将页面里的header，footer或者sidebar单独写成一个文件。这些partial的模板要放在`layout/partials`下面。例如，你有一个`header.html`的partial模板：
```html
<h1>{{.Title}}</h1>
<p>{{.Date}}</p>
```
然后在single template里，你想要用这个header模板,就要使用partial这个函数，`{{ partial "header" .}}`, 这里的点代表你想要使用的范围是所有，这样你才可以访问到partial里所有的变量。
```html
{{ partial "header" . }}
{{.Title}}
```

这是一个`baseof.html`模板
```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    Top of baseof
    <hr>
    {{ block "main" . }}

    {{ end }}
    <hr>
    bottom of baseof
  </body>
</html>
```
在`single.html`模板中定义对应的main block：
```html
{{ define "main" }}
 this is the single template
 add any content here
{{ end }}
```
你可以在baseof里写好通用的block，也可以在单个模板里重写，但是这个功能大大减少了代码的冗余。

## Variable
在之前已经有在html里调用变量的例子了，基本上就是用双花括号。例如要在html中插入标题和日期：
```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title>{{ .Title }}</title>
  </head>
  <body>
    Today is {{ .Date }}
    This is the link to the file {{ .URL }}
  </body>
</html>
```
在front matter可以有更多的自定义变量，通过`{{ .Param.Var }}`访问。除了在front matter 里定义，你可以在html里直接创建变量。
```html
{{ $myVar := "aString" }}
<h1>This is a custom var: {{ $myVar }}</h1>
```
Hugo还有很多特有的变量可以在html里调用，具体有哪些变量可以在[官方文档](https://gohugo.io/categories/variables-and-params)里看到。

## Function
Hugo预先写好了一些函数方便使用，函数只能在layout下的模板文件里使用。用法也很简单:
```html
{{ funcName param1 param2 .. paramN}}
```
有一些处理字符串的函数比较常用，如truncate会截断字符串到指定长度：
```html
{{ truncate 10 "This a way long long long long way to go" }}
```
hugo也可以写循环：
```html
{{ range .Pages }} #.Pages 是一个hugo变量，代表当前section的page数。
 {{ .Title }} #这里打印出所有文章
{{ end }}
```
hugo所有的函数也可以在[这里](https://gohugo.io/functions/)查到。

当然，hugo也可以有if-else，也是需要放在双括号里。
```html
{{ $a := "apple" }}
{{ $b := "orange" }}

{{ if not (eq $a $b) }}
 True
{{ else }}
 False
{{ end }}
```
这个条件执行语句可以用在style里，例如你列出了所有的文件，但是想要当前标黄页面名。结合for loop和if：
```html
<h1>Single template</h1>

{{ $title := .Title}} #当前页面名
{{ range .Site.Pages }}
<li><a href="{{ .URL }}" style="{{if eq $title .Title }} background-color: yellow; {{end}}"></a>{{.Title}}</li>
```

## Data files
在根目录下，我还看见了一个data文件夹，是空的。我不太懂怎么用，通常动态网页会有一个对应的数据库，但是hugo用于生产静态网页，data文件夹放的都是YAML,TOML或者JSON格式的文件。例如你存了一些JSON格式表格信息，可以通过for loop在模板里访问到这个文件通过变量{{ .Site.Data.filename }}。这个应该很少用的。


## Shortcodes
shortcodes算是hugo一个比较高级的功能了，我应该不怎么用到。markdown来写文档很简单，如果要在页面嵌入像视频这样的复杂的元素就需要在markdown中插入一大段html。这样看起来并不美观，而且markdown并不支持和html共存，需要进行解析。shortcodes只是用很小一段代码标记，就可以在网页生成的时候，用对应的html代码去替换这个区域。具体的html模板都是放在`layout/shortcodes/`下。

比如说建立了一个youtube嵌入视频的hmtl模板`layout/shortcodes/youtube.html`
```html
<style>
    #biliplayer {
      width: 100%;
      height: 600px;
    }
    @media only screen and (min-device-width: 320px) and (max-device-width: 480px) {
      #biliplayer {
        width: 100%;
        height: 250px;
      }
    }
</style>

<div class="embed video-player">
<iframe class="youtube-player" type="text/html" width="640" height="385" src="https://www.youtube.com/embed/{{ index .Params 0 }}" allowfullscreen frameborder="0">
</iframe>
</div>
```
在markdown文档中，只用调用这个模板并提供对应视频的AV号就可以了。

```「markdown」

## here is a youtube video.
{{</* youtube r_Ws-jW1Rz4 */>}}

```

借用一下同学的视频。
{{< youtube r_Ws-jW1Rz4 >}}

简单的想，shortcodes给我的感觉有点像自定义的函数，然后可以在markdown里直接调用。例如你可以写一个`myshortcode.html`在shortcodes文件夹下：

```html
 <p style="color:{{index .Params 0}}">This is a string that can be any color specified by shortcodes</p>
```

然后在md文件中调用它：
```{markdown}
{{</* myshortcode brown */>}}
```
也可以不用参数传递的形式传入参数，可以用shortcodes的标签包括内容。

```html
{{</* myshortcode */>}}
anytext that are not been rendered.
{{ /myshortcode}}
```
如果你需要传入的字符被render:
```html
{{%/* myshortcode */%}}
**bold text**
{{ /myshortcode}}
```
我没有很深入的研究更多的用法，希望以后有机会可以用到。

## Go public
使用`hugo server`命令可以本地预览网页，最终还是要生成一堆html用于放在自己的或者github的服务器上。只需要运行`hugo`，会生成一个public文件夹包含所有的html。把这个文件夹上传至服务器就可以运行了。我是放在github上的，我已经有一个github page的仓库.同时我也想把这个hugo的内容也上传到github。可以通过`git submodule`将生成的public子文件夹作为`username.github.io`这个远程项目，首先把public文件夹删除。

```bash
git submodule add https://github.com/bakeronit.github.io public
```
这时目录下会产生一个`.gitmodules`文件：
```TOML
[submodule "public"]
	path = public
	url = https://github.com/bakeronit/bakeronit.github.io.git
```
然后可以创建一个`deploy.sh`文件：
```bash
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public
# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back up to the Project Root
cd ..
```
直接运行这个脚本就可以了，其实主题也可以通过submodule。。但是要注意删除`submodule`需要手动删除，不能直接`rm`。
接下来我想把hugo源文件也上传到github，这时候我不想要把public也提交上去，就要用到`.gitignore`配置文件，在里面写入不想上传的文件名。
```
.DS_Store
/public
```
然后把这个项目推送到github，中间发生一个小插曲，原来最近github把默认的branch名改成了`main`，以前是`master`。现在要`push`到`main branch`了。

## Cusom domain
如果你想要用自己的域名，只需要在static文件夹下写一个CNAME文件，内容是自己的域名。这样每次hugo都会将CNAME拷贝到网页项目的根目录，然后就可以通过你的域名访问这个github page了。不过首先不要忘了在域名解析服务商那里添加几条github的记录，我几年前用的dnspod，现在把找回来居然是腾讯云了，它有A记录的数量限制。github有4条A记录，最后我直接在godaddy上搞了。
```
#A record
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```
检查是否解析成功了:
```
> dig EXAMPLE.COM +noall +answer
EXAMPLE.COM     3600    IN A     185.199.108.153
EXAMPLE.COM     3600    IN A     185.199.109.153
EXAMPLE.COM     3600    IN A     185.199.110.153
EXAMPLE.COM     3600    IN A     185.199.111.153
```
另外，如果有强迫症想要一个绿🔐，github现在提供Let's Encrypt免费认证了。其实具体是什么我也不懂。但是只用在网页项目的配置页面勾选Enforce HTTPS就好了，有可能要等一段时间才行，我睡了一觉起来就好了。
