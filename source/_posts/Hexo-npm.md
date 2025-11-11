---
title: Hexo各种优化
date: '2020-03-14 11:00'
summary: 在国内npm的下载速度着实是有点慢。由于下面内容会安装较多的插件，建议先更改npm仓库地址，以便能更快的安装插件
categories: 网站优化
tags:
  - npm
  - hexo
abbrlink: 632e
---

## 0x01 npm速度优化

------

> 在国内npm的下载速度着实是有点慢。由于下面内容会安装较多的插件，建议先更改npm仓库地址，以便能更快的安装插件

### 使用淘宝镜像

- npm的默认仓库地址是 `https://registry.npmjs.org/`

- 可以使用以下命令查看当前npm的仓库地址

```sh
npm config get registry
```

- 可以使用以下命令来改变默认下载地址，从而达到不安装`cnpm`就能采用淘宝镜像的目的，然后使用上面的get命令查看是否设置成功。

```sh
npm config set registry https://registry.npm.taobao.org
```

### 安装CNPM

- 安装cnpm，命令：

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

- 安装后，使用以下命令测试是否安装成功：

```bash
cnpm -v
```

- 成功后，以后都使用 `cnpm` 代替以前 `npm` 来执行命令！

## 0x02 访问速度优化

### 图片加载优化

什么是`预加载` 和 `懒加载`

`预加载`就是进入项目前提前加载资源，避免在项目中加载缓慢，影响用户体验

- 缺点：会增加服务器压力

`懒加载`一般是当图片滚动进可视窗口内才加载图片，可视窗口之外的图片则不加载

- 优点：对服务器有一定的缓解压力作用

#### 懒加载法

装图片懒加载插件： [hexo-lazyload-image](https://github.com/Troy-Yang/hexo-lazyload-image)

在Hexo根目录执行

```bash
npm install hexo-lazyload-image --save
```

然后在Hexo配置文件末尾加入以下代码

```yml
lazyload:
  enable: true 
  onlypost: false  # 是否只对文章的图片做懒加载
  loadingImg: # eg ./images/loading.gif
```

到这里就配置完了，执行`hexo cl&&hexo g&&hexo s`就有效果了，以后博客上的图片就都是懒加载了，以上步骤理论上任何主题都可以用

一般情况下懒加载会和gallery插件会发生冲突，结果可能就是点开图片，左翻右翻都是`loading image。matery`主题的解决方案是：

修改 `/themes/matery/source/js` 中的 `matery.js`文件

在第108行加上：

```txt
$(document).find('img[data-original]').each(function(){
    $(this).parent().attr("href", $(this).attr("data-original"));
});
```

做完这步之后，还有点小Bug，首页的logo点击会直接打开logo图，而不是跳到首页。

伪解决方案：打开 `/themes/matery/layout/_partial/header.ejs`文件，

在`img`和`span`的两个头加个`div`：

```html
<div class="brand-logo">
    <a href="<%- url_for() %>" class="waves-effect waves-light">
        <div>
            <% if (theme.logo !== undefined && theme.logo.length > 0) { %>
            <img src="<%= theme.logo %>" class="logo-img" alt="LOGO">
            <% } %>
            <span class="logo-span"><%- config.title %></span>
        </div>
    </a>
</div>
```

#### 自定义loading图片

`hexo-lazyload-image` 插件提供了自定义loading图片的选项

方法就是在 `loadingImg` 后配置图片的路径就好了

```yaml
lazyload:
  enable: true 
  onlypost: false  # 是否只对文章的图片做懒加载
  loadingImg: /medias/loading.gif # eg ./images/loading.gif
```

#### 懒加载优化

> 经过以上操作就已经很完美了，以下内容可做可不做

- 其实第一次加载后本地都是有缓存的，如果每次都把loading显示出来就不那么好看

- 所以我们需要对插件进行魔改，让图片稍微提前加载，避开加载动画

- 打开 `Hexo根目录`>`node_modules` > `hexo-lazyload-image` > `lib` > `simple-lazyload.js` 文件

- 第9行修改为：

```js
&& rect.top <= (window.innerHeight +240 || document.documentElement.clientHeight +240)
```

>  作用：提前240个像素加载图片；当然这个值也可以根据自己情况修改

### 代码压缩优化

#### gulp实现代码压缩

- cd到Hexo根目录依次执行以下命令：

```bash
# 全局安装gulp模块
npm install gulp -g
# 安装各种小功能模块  执行这步的时候，可能会提示权限的问题，最好以管理员模式执行
npm install gulp gulp-htmlclean gulp-htmlmin gulp-minify-css gulp-uglify gulp-imagemin --save
# 额外的功能模块
npm install gulp-debug gulp-clean-css gulp-changed gulp-if gulp-plumber gulp-babel babel-preset-es2015 del @babel/core --save
```

- 在Hexo根目录新建文件 `gulpfile.js`，并复制以下内容到文件中，有中文注释，可以根据自己需求修改。

```txt
var gulp = require("gulp");
var debug = require("gulp-debug");
var cleancss = require("gulp-clean-css"); //css压缩组件
var uglify = require("gulp-uglify"); //js压缩组件
var htmlmin = require("gulp-htmlmin"); //html压缩组件
var htmlclean = require("gulp-htmlclean"); //html清理组件
var imagemin = require("gulp-imagemin"); //图片压缩组件
var changed = require("gulp-changed"); //文件更改校验组件
var gulpif = require("gulp-if"); //任务 帮助调用组件
var plumber = require("gulp-plumber"); //容错组件（发生错误不跳出任务，并报出错误内容）
var isScriptAll = true; //是否处理所有文件，(true|处理所有文件)(false|只处理有更改的文件)
var isDebug = true; //是否调试显示 编译通过的文件
var gulpBabel = require("gulp-babel");
var es2015Preset = require("babel-preset-es2015");
var del = require("del");
var Hexo = require("hexo");
var hexo = new Hexo(process.cwd(), {}); // 初始化一个hexo对象
  
  // 清除public文件夹
  gulp.task("clean", function () {
      return del(["public/**/*"]);
  });
  
  // 下面几个跟hexo有关的操作，主要通过hexo.call()去执行，注意return
  // 创建静态页面 （等同 hexo generate）
  gulp.task("generate", function () {
      return hexo.init().then(function () {
          return hexo
              .call("generate", {
                  watch: false
              })
              .then(function () {
                  return hexo.exit();
              })
              .catch(function (err) {
                  return hexo.exit(err);
              });
      });
  });
  
  // 启动Hexo服务器
  gulp.task("server", function () {
      return hexo
          .init()
          .then(function () {
              return hexo.call("server", {});
          })
          .catch(function (err) {
              console.log(err);
          });
  });
  
  // 部署到服务器
  gulp.task("deploy", function () {
      return hexo.init().then(function () {
          return hexo
              .call("deploy", {
                  watch: false
              })
              .then(function () {
                  return hexo.exit();
              })
              .catch(function (err) {
                  return hexo.exit(err);
              });
      });
  });
  
  // 压缩public目录下的js文件
  gulp.task("compressJs", function () {
      return gulp
          .src(["./public/**/*.js", "!./public/libs/**"]) //排除的js
          .pipe(gulpif(!isScriptAll, changed("./public")))
          .pipe(gulpif(isDebug, debug({ title: "Compress JS:" })))
          .pipe(plumber())
          .pipe(
              gulpBabel({
                  presets: [es2015Preset] // es5检查机制
              })
          )
          .pipe(uglify()) //调用压缩组件方法uglify(),对合并的文件进行压缩
          .pipe(gulp.dest("./public")); //输出到目标目录
  });
  
  // 压缩public目录下的css文件
  gulp.task("compressCss", function () {
      var option = {
          rebase: false,
          //advanced: true, //类型：Boolean 默认：true [是否开启高级优化（合并选择器等）]
          compatibility: "ie7" //保留ie7及以下兼容写法 类型：String 默认：''or'*' [启用兼容模式； 'ie7'：IE7兼容模式，'ie8'：IE8兼容模式，'*'：IE9+兼容模式]
          //keepBreaks: true, //类型：Boolean 默认：false [是否保留换行]
          //keepSpecialComments: '*' //保留所有特殊前缀 当你用autoprefixer生成的浏览器前缀，如果不加这个参数，有可能将会删除你的部分前缀
      };
      return gulp
          .src(["./public/**/*.css", "!./public/**/*.min.css"]) //排除的css
          .pipe(gulpif(!isScriptAll, changed("./public")))
          .pipe(gulpif(isDebug, debug({ title: "Compress CSS:" })))
          .pipe(plumber())
          .pipe(cleancss(option))
          .pipe(gulp.dest("./public"));
  });
  
  // 压缩public目录下的html文件
  gulp.task("compressHtml", function () {
      var cleanOptions = {
          protect: /<\!--%fooTemplate\b.*?%-->/g, //忽略处理
          unprotect: /<script [^>]*\btype="text\/x-handlebars-template"[\s\S]+?<\/script>/gi //特殊处理
      };
      var minOption = {
          collapseWhitespace: true, //压缩HTML
          collapseBooleanAttributes: true, //省略布尔属性的值 <input checked="true"/> ==> <input />
          removeEmptyAttributes: true, //删除所有空格作属性值 <input id="" /> ==> <input />
          removeScriptTypeAttributes: true, //删除<script>的type="text/javascript"
          removeStyleLinkTypeAttributes: true, //删除<style>和<link>的type="text/css"
          removeComments: true, //清除HTML注释
          minifyJS: true, //压缩页面JS
          minifyCSS: true, //压缩页面CSS
          minifyURLs: true //替换页面URL
      };
      return gulp
          .src("./public/**/*.html")
          .pipe(gulpif(isDebug, debug({ title: "Compress HTML:" })))
          .pipe(plumber())
          .pipe(htmlclean(cleanOptions))
          .pipe(htmlmin(minOption))
          .pipe(gulp.dest("./public"));
  });
  
  // 压缩 public/medias 目录内图片
  gulp.task("compressImage", function () {
      var option = {
          optimizationLevel: 5, //类型：Number 默认：3 取值范围：0-7（优化等级）
          progressive: true, //类型：Boolean 默认：false 无损压缩jpg图片
          interlaced: false, //类型：Boolean 默认：false 隔行扫描gif进行渲染
          multipass: false //类型：Boolean 默认：false 多次优化svg直到完全优化
      };
      return gulp
          .src("./public/medias/**/*.*")
          .pipe(gulpif(!isScriptAll, changed("./public/medias")))
          .pipe(gulpif(isDebug, debug({ title: "Compress Images:" })))
          .pipe(plumber())
          .pipe(imagemin(option))
          .pipe(gulp.dest("./public"));
  });
  // 执行顺序： 清除public目录 -> 产生原始博客内容 -> 执行压缩混淆 -> 部署到服务器
  gulp.task(
      "build",
      gulp.series(
          "clean",
          "generate",
          "compressHtml",
          "compressCss",
          "compressJs",
          "compressImage",
          gulp.parallel("deploy")
      )
  );
  
  // 默认任务
  gulp.task(
      "default",
      gulp.series(
          "clean",
          "generate",
          gulp.parallel("compressHtml", "compressCss", "compressJs","compressImage")
      )
  );
  //Gulp4最大的一个改变就是gulp.task函数现在只支持两个参数，分别是任务名和运行任务的函数
```

- 以后的执行方式有两种：

  1. 直接在Hexo根目录执行 `gulp`或者 `gulp default` ，这个命令相当于 `hexo cl&&hexo g` 并且再把代码和图片压缩。
  2. 在Hexo根目录执行 `gulp build` ，这个命令与第1种相比是：在最后又加了个 `hexo d` ，等于说生成、压缩文件后又帮你自动部署了。

- 值得注意的是：这个加入了图片压缩，如果不想用图片压缩可以把第154行的 `"compressImage",` 和第165行的 `,"compressImage"` 去掉即可

#### hexo-neat插件实现代码压缩

- 可能以上方法比较复杂，来介绍个简单的，[hexo-neat](https://github.com/rozbo/hexo-neat)插件也是用来压缩代码的，底层也是通过gulp来实现的。

- 但是这个插件是有Bug的：

  - 压缩 md 文件会使 markdown 语法的代码块消失
  - 会删除全角空格

- Hexo根目录执行安装代码：

```bash
npm install hexo-neat --save
```

- 在Hexo配置文件`_config.yml` 末尾加入以下配置：

```yml
  neat_enable: true
  neat_html:
    enable: true
    exclude:
  neat_css:
    enable: true
    exclude:
      - '*.min.css'
  neat_js:
    enable: true
    mangle: true
    output:
    compress:
    exclude:
      - '*.min.js'
```

- 然后直接 `hexo cl&&hexo g` 就可以了，会自动压缩文件 。

- **补充**：为了解决以上Bug，**对于matery主题**（其他主题自行解决）需要将以上默认配置修改为：

```yml
  #hexo-neat 优化提速插件（去掉HTML、css、js的blank字符）
  neat_enable: true
  neat_html:
    enable: true
    exclude:
      - '**/*.md'
  neat_css:
    enable: true
    exclude:
      - '**/*.min.css'
  neat_js:
    enable: true
    mangle: true
    output:
    compress:
    exclude:
      - '**/*.min.js'
      - '**/**/instantpage.js'
      - '**/matery.js'
```

### 全站CDN加速

放在Github的资源在国内加载速度比较慢，因此需要使用CDN加速来优化网站打开速度，[jsDelivr](https://www.jsdelivr.com/) + Github便是免费且好用的CDN，非常适合博客网站使用。

**用法：**

```http
https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名@发布的版本号/文件路径
```

**例如：**

```http
https://cdn.jsdelivr.net/gh/jeremysvn/muyun/medias/loading.gif
```

注意：版本号不是必需的，是为了区分新旧资源，如果不使用版本号，将会直接引用最新资源

还可以配合`PicGo`图床上传工具的**自定义域名前缀**来上传图片，使用极其方便。

### 向百度推送自己的资源

##### 使用sitemap方式推送

**安装sitemap插件**

```bash
npm install hexo-generator-sitemap --save 
npm install hexo-generator-baidu-sitemap --save
```

这两个插件是用来生成 `Sitemap文件` 的插件，而 `Sitemap` 是用来告知搜索引擎我们的网站上有哪些可供抓取的网页的。

**注意一点：**

> hexo配置文件中的url一定要改成你的域名，这两个插件是根据你的url生成站点地图的。

安装后直接执行`hexo cl&&hexo g`命令，然后就会在网站根目录生成`sitemap.xml`文件和`baidusitemap.xml文件`，其中`sitemap.xml`文件是搜索引擎通用的文件，`baidusitemap.xml`是百度专用的`sitemap`文件。

有`sitemap文件`之后，再将生成的`sitemap文件`提交给百度或者其他搜索引擎

百度方式：在自动提交的sitemap那里填写自己`sitemap文件`的URL地址即可

```http
https://你的域名/baidusitemap.xml
```

##### 自动推送方式

只要每个需要被百度爬取的HTML页面中加入一段JS代码即可：

```txt
<script>
(function(){
    var bp = document.createElement('script');
    var curProtocol = window.location.protocol.split(':')[0];
    if (curProtocol === 'https') {
        bp.src = 'https://zz.bdstatic.com/linksubmit/push.js';
    }
    else {
        bp.src = 'http://push.zhanzhang.baidu.com/push.js';
    }
    var s = document.getElementsByTagName("script")[0];
    s.parentNode.insertBefore(bp, s);
})();
</script>
```

我所使用的matery主题可以自动给每个页面加上这段代码，只需在主题配置文件中配置：

```yml
# 百度搜索资源平台提交链接
baiduPush: true
```

即可！

其他主题一般都有这个功能的实现，如果没有的话，想办法在每个页面加入以上JS代码即可，原理是一样。

##### 主动推送方式

安装主动推送插件：[hexo-baidu-url-submit](https://github.com/huiwang/hexo-baidu-url-submit)

```bash
npm install hexo-baidu-url-submit --save
```

然后打开`hexo配置文件`，在末尾加入以下配置：

```yml
# hexo-baidu-url-submit  百度主动推送
baidu_url_submit:
  count: 80 # 提交最新的一个链接
  host: blog.sky03.cn # 在百度站长平台中注册的域名
  token: xxxxxxx # 请注意这是您的秘钥， 所以请不要把博客源代码发布在公众仓库里!
  path: baidu_urls.txt # 文本文档的地址， 新链接会保存在此文本文档里
```

密匙的获取是在百度的自动提交的主动推送那里。

再加入新的`deploy`：

```yml
deploy:
- type: baidu_url_submitter
```

这样每次执行 `hexo d` 的时候，新的链接就会被推送了。
推送成功时,会有如下终端提示!

一般来说，推送失败基本都是地址不相符造成的，我们只需对比`baidu_url_submit`在`public`中生成的`baidu_urls.txt`的地址,与自己填写在`host`字段中的地址对比看是否一样即可。

### 提交 robots.txt

`robots.txt` 是一种存放于网站根目录下的 `ASCII` 编码的文本文件，它的作用是告诉搜索引擎此网站中哪些内容是可以被爬取的，哪些是禁止爬取的。

每个人站点目录可能不太一样，可以参考下我的 `robots.txt` 文件，内容如下：

```yaml
User-agent: *
Allow: /
Allow: /posts/
Disallow: /about/
Disallow: /archives/
Disallow: /js/
Disallow: /css/
Disallow: /contact/
Disallow: /fonts/
Disallow: /friends/
Disallow: /libs/
Disallow: /medias/
Disallow: /page/
Disallow: /tags/
Disallow: /categories/
```

编写完以上内容再重新部署一下，然后到百度资源平台的`数据监控`->`Robots`点击`检测并更新` 看能不能检测到。

同样注意：刚添加的站点没有进行 `HTTPS认证`，直接检测有可能会报301错误。

### 配置 Nofollow

- nofollow 是HTML页面中 `a标签` 的 属性值。
- 这个属性的作用是：告诉搜索引擎的爬虫不要追踪该链接，为了对抗博客垃圾留言信息

### URL优化

一般来说，SEO搜索引擎优化认为，网站的最佳结构是 **用户从首页点击三次就可以到达任何一个页面**，但是我们使用`Hexo`编译的站点结构的`URL`是：`域名/年/月/日/文章标题`四层的结构，这样的`URL`结构很不利于`SEO`，爬虫就会经常爬不到我们的文章，于是，我们需要优化一下网站文章的`URL`

**方案一**：

直接改成`域名/文章标题`的形式，在`Hexo配置文件`中修改`permalink`如下：

```yml
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://baize.cc
root: /
permalink: :title.html
permalink_defaults:
```

**这个方式有个不好的地方：**

直接以`文章的标题`作为URL，而我们所写的文章的标题一般都是中文，但是URL只能用字母数字和标点符号表示，所以中文的URL只能被转义成一堆符号，而且还特别长。

**方案二**：

安装固定链接插件：[hexo-abbrlink](https://github.com/rozbo/hexo-abbrlink)

插件作用：自动为每篇文章生成一串数字作每篇文章的URI地址。每篇文章的`Front-matter`中会自动增加一个配置项：`abbrlink: xxxxx`，该项的值就是当前文章的URI地址。

1. Hexo根目录执行：

```bash
npm install hexo-abbrlink --save
```

2. `Hexo配置文件`末尾加入以下配置：

```yml
# hexo-abbrlink config 、固定文章地址插件
abbrlink:
   alg: crc16  #算法选项：crc16、crc32，区别见之前的文章，这里默认为crc16丨crc32比crc16复杂一点，长一点
   rep: dec    #输出进制：十进制和十六进制，默认为10进制。丨dec为十进制，hex为十六进制
```

3. `Hexo配置文件`中修改`permalink`如下：

```yml
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://baize.cc
root: /
permalink: posts/:abbrlink.html
permalink_defaults:
```

这样站点结构就变成了：`域名/posts/xxx.html`

