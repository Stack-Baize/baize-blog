---
title: Hexo版本升级指南
date: '2021-02-09 11:00'
summary: 更换了新主题，在hexo g和hexo s的时候总会报如下错误，在主题github issues逛了一圈后，感觉应该是我的hexo版本太老了。
categories: 网站优化
tags:
  - npm
  - hexo
abbrlink: 15ac
---


## 前言

更换了新主题，在hexo g和hexo s的时候总会报如下错误，在主题github issues逛了一圈后，感觉应该是我的hexo版本太老了。



```shell
➜ hexo s
(node:64285) [DEP0061] DeprecationWarning: fs.SyncWriteStream is deprecated.
INFO  Start processing
ERROR Process failed: layout/archive.ejs
SyntaxError: Invalid or unexpected token in "/Users/lanvnal/Files/blog/themes/hexo-theme-matery/layout/archive.ejs"
    at new Function (<anonymous>)
    at Object.exports.compile (/Users/lanvnal/Files/blog/node_modules/ejs/lib/ejs.js:242:14)
    at Function.ejsRenderer.compile (/Users/lanvnal/Files/blog/node_modules/hexo-renderer-ejs/lib/renderer.js:11:14)
    at Theme._View.View.View._precompile (/Users/lanvnal/Files/blog/node_modules/hexo/lib/theme/view.js:117:31)
```

决定升级hexo版本，但是没找到很明确的升级hexo的文章，遂做一下记录。

## 升级

1、全局升级hexo-cli，先`hexo version`查看当前版本，然后`npm i hexo-cli -g`，再次`hexo version`查看是否升级成功。

```bash
npm i hexo-cli -g
npm update
hexo version
```



2、使用`npm install -g npm-check`和`npm-check`，检查系统中的插件是否有升级的，可以看到自己前面都安装了那些插件

```bash
npm install -g npm-check
npm-check
```



3、使用`npm install -g npm-upgrade`和`npm-upgrade`，升级系统中的插件

```bash
npm install -g npm-upgrade
npm-upgrade
```

4、使用`npm update -g`和`npm update --save`

更新全局包与更新生产环境依赖包

```bash
npm update <name> -g

npm update <name> --save
```

save参数：`npm install X –save`:

- 会把X包安装到`node_modules`目录中

- 会在`package.json`的dependencies属性下添加X

- 之后运行`npm install`命令时，会自动安装X到node_modules目录中

如果不加save参数的话，之后把X包安装到node_modules目录中，不会添加到dependencies文件中。再查看hexo文件夹下面的dependencies文件,可以看到hexo的版本已经更新了。

```bash
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.7.1"
  },
  "dependencies": {
    "hexo": "^3.7.1",
    "hexo-deployer-git": "^0.3.1",
    "hexo-fs": "^0.2.3",
    "hexo-generator-archive": "^0.1.5",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-index": "^0.2.1",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.3.1",
    "hexo-renderer-marked": "^0.3.2",
    "hexo-renderer-stylus": "^0.3.3",
    "hexo-server": "^0.3.2"
  }
}
```



PS：第四步遇到了错误，错误提示如下：



```shell
> fsevents@1.2.11 install /Users/lanvnal/Files/blog/node_modules/hexo/node_modules/fsevents
> node-gyp rebuild

xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance

xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance

No receipt for 'com.apple.pkg.CLTools_Executables' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLILeo' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLI' found at '/'.
gyp: No Xcode or CLT version detected!
```

其实已经安装过了xcode cli，但是这里还是报错了，估计和苹果新系统有关，重装就好了，操作如下：

如果像以前一样执行`xcode-select --install`会有如下报错：



```bash
xcode-select: error: command line tools are already installed, use "Software Update" to install updates
```

解决办法：



```shell
sudo rm -rf /Library/Developer/CommandLineTools
xcode-select --install
```

然后再执行第四步，完美升级。