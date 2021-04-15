  之前一直觉得插件开发很难，迟迟不敢接触，但每次看到或者用别人开发的插件时，又非常羡慕，这次刚好有个项目开需要开发 vscode 插件，借此机会终于要对 vscode 插件下手了。本篇文章主要包含入门开发和过程中遇到的一些问题，希望能给大家带来帮助，避免跟我一样踩坑。下面就具体的来看一下吧：

## 安装脚手架

使用官方脚手架生成项目，要先安装脚手架，命令如下：

```
npm install -g yo generator-code
```

## 生成项目

切到对应目录，执行 yo code，将会出现如下图的一个选择，根据自己的需求选择对应类型的扩展模板，剩下的就根据提示一步步操作即可，运行完后就生成了一个 vscode 插件项目。

![image-20210402094946242](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ce5bd8e9b2d40f7add42b8f74e3c32e~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf17001c8151426dbcbd830b72f3e93e~tplv-k3u1fbpfcp-zoom-1.image)

## 项目核心结构

​     执行 yo code 生成项目后，仔细看下项目结构，我们比较重要的其实就两个文件，分别是 package.json 和 extension.js。package.json 是整个插件工程的配置文件，extension.js 是工程的入口文件，下面我们详细介绍一下这两个文件。

### pacgage.json

下面是 package.json 文件的常用配置：

```javascript
{
  // 插件名称,需用全小写无空格的字母组成
  "name": "vue-typescript-snippets-plugin",
  // 插件市场显示的插件名称
  "displayName": "vue-typescript-snippets-plugin",
  // 简单描述插件的功能
  "description": "生成 ts 的 vue 代码片段",
  "version": "0.0.1",
  // 插件支持的 vscode 最低版本
  "engines": {
     "vscode": "^1.55.0"
  },
  // 插件的应用市场分类，可选值包括 [Programming Languages, Snippets, Linters, Themes, Debuggers,Formatters, Keymaps, SCM Providers, Other, Extension Packs, Language Packs]
  "categories": [
     "Other"
  ],
  // 扩展的激活事件数组
  "activationEvents": [
     "onCommand:vue-typescript-snippets-plugin.helloWorld"
  ],
  // 插件入口
  "main": "./dist/extension.js",
  // 配置内容项
  "contributes": {
      // 命令
      "commands": [
	{
	  "command": "vue-typescript-snippets-plugin.helloWorld",
	  "title": "Hello World"
	}
      ]
  },
  "scripts": {
      "vscode:prepublish": "yarn run package",
      "compile": "webpack",
      "watch": "webpack --watch",
      "package": "webpack --mode production --devtool hidden-source-map",
      "test-compile": "tsc -p ./",
      "test-watch": "tsc -watch -p ./",
      "pretest": "yarn run test-compile && yarn run lint",
      "lint": "eslint src --ext ts",
      "test": "node ./out/test/runTest.js"
  },
  "devDependencies": {
      "@types/vscode": "^1.55.0",
      "@types/glob": "^7.1.3",
      "@types/mocha": "^8.0.4",
      "@types/node": "^12.11.7",
      "eslint": "^7.19.0",
      "@typescript-eslint/eslint-plugin": "^4.14.1",
      "@typescript-eslint/parser": "^4.14.1",
      "glob": "^7.1.6",
      "mocha": "^8.2.1",
      "typescript": "^4.1.3",
      "vscode-test": "^1.5.0",
      "ts-loader": "^8.0.14",
      "webpack": "^5.19.0",
      "webpack-cli": "^4.4.0"
  }
}

```

这个就是我们初始化生成的 package.json 文件，其中比较核心的是 [`activationEvents`](https://liiked.github.io/VS-Code-Extension-Doc-ZH/#/extensibility-reference/activation-events) 和 [`contributes`](https://liiked.github.io/VS-Code-Extension-Doc-ZH/#/extensibility-reference/contribution-points)

#### activationEvents

activationEvents 是配置激活事件的，我们插件一开始是没有被激活的，只有当触发了 activationEvents 中配置的事件，插件才会被激活，激活的事件列表如下：

- onLanguage: 打开指定语言文件时会激活

  ```json
  "activationEvents": [
   	"onLanguage:html"
   ]
  ```

- onCommand: 调用命令时会激活

  ```json
  "activationEvents": [
    "onCommand:extension.sayHello"
  ]
  ```

- onDebug: 调试会话启动前会激活

  ```json
  "activationEvents": [
    "onDebug"
  ]
  ```

- workspaceContains: 工作区有包含符合 glob模式的文件时会激活

  ```json
  "activationEvents": [
    "workspaceContains:**/.editorconfig"
  ]
  ```

- onFileSystem:  以协议打开文件或文件夹时会激活

  ```json
  "activationEvents": [
    "onFileSystem:sftp"
  ]
  ```

- onView:  指定 id 的视图展开时会激活

  ```json
  "activationEvents": [
    "onView:nodeDependencies"
  ]
  ```

- onUri:  插件的系统级 URI 打开时会激活，这个URI 协议需要带上vscode或者vscode-insiders协议。

  ```json
  "activationEvents" [
    "onUri"
  ]
  ```

- onWebviewPannel:  当 vscode 需要恢复一个符合 viewType 的 webview 时会激活。

  ```json
  "activationEvents": [
    "onWebviewPanel:catCoding"
  ]
  ```

- onCustomEditor: 当 vscode 需要创建一个符合 viewType 的自定义编辑器时会激活。

  ```json
  "activationEvents": [
    "onCustomEditor:catCustoms.pawDraw"
  ]
  ```

- onStartupFinished: 在 vscode 启动后一段时间会激活，类似于”*“， 但是不会减慢 vscode 的启动速度。

  ```json
  "activationEvents": [
    "onStartupFinished"
  ]
  ```

- *:  vscode启动时会激活，为确保用户体验，请在没有其他激活事件时使用。

  ```json
  "activationEvents": [
    "*"
  ]
  ```



#### contributes

还有一个比较核心的配置是contributes，它被称为 Contribution Points，可以用来注册各种配置项来扩展 vscode 的能力。下面是目前可用的配置项列表：

> configuration：配置的内容会暴露给用户，可以从”用户设置“和”工作区设置“中修改你暴露的选项。
>
> configurationDefaults：为特定语言配置编辑器的默认值。
>
> commands：设置命令标题和命令体，会展示在命令面板（⇧⌘P）。
>
> menus：为编辑器或文件管理器设置命令的菜单项。至少包含菜单显示的时机和选择菜单执行的命令。
>
> keybindings：配置快捷键
>
> languages：配置语言，引入新的语言或加强 vscode 已有的语言支持。
>
> debuggers：配置 vscode 调试器。
>
> breakpoints：配置哪些语言可以设置断点。
>
> grammars：为一门语言配置 TextMate 语法
>
> themes：为 vscode 增加主题
>
> iconThemes：配置 vscode 的文件图标主题
>
> productIconThemes：配置 vscode 的产品图标主题
>
> snippets：为一门语言增加代码片段
>
> jsonValidation：配置 json 文件的校验器
>
> views：为 vscode 增加视图
>
> viewsWelcome：配置自定义视图的欢迎内容
>
> viewsContainers：配置自定义视图的视图容器
>
> problemMatchers：配置问题定位器的模式
>
> problemPatterns：配置可以在问题定位器(见上)中可以使用的模式名称
>
> taskDefinitions：配置和定义一个 object 结构，定义系统中唯一的配置任务
>
> colors：配置可用于编辑器装饰器和状态栏的颜色
>
> typescriptServerPlugins：配置 vscode 的 js 和 ts 支持的 Typescript 服务器插件
>
> resourceLabelFormatters: 配置资源标签格式化程序，它将指定如何在工作台中的任何地方显示 URI



### extension.js 

&nbsp;&nbsp; 
extension.js 是插件的入口文件，会导出两个函数：activate 和的activate，当注册的激活事件被触发时，即会执行 activate 中的代码，deactivate 则提供了插件关闭前执行清理工作的机会。由于这里的代码只会被执行一次，所以需要在 activate 中使用 registerCommand 来注册命令，这里的参数必须与 package.json 中定义的  一致，下面是简单的示例：

![image-20210402174828752](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da44effea5e749e3b6d4d845a1f7d011~tplv-k3u1fbpfcp-zoom-1.image)

&nbsp;&nbsp; 
registerCommand的第二个参数匿名函数中的代码，在每次该命令被触发时都会执行一次。我们在vscode 中启动调试之后，会看到弹出一个新的窗口，在新窗口按下 command+shift+p 即可调用插件，输入我们定义的 Hello world 会触发上面匿名函数内的代码，启动我们的插件，在 vscode 的右下角出现一个通知框，这就表示我们的插件启动成功了，接下来我们就可以根据需求开发自己想要的功能了。

## 运行调试

我们在插件开发过程中，如何实时看到自己插件的效果呢，vscode 提供给我们了，按下图中的顺序操作，1 将切到 debug 的操作面板，点击 2 会开启一个新的窗口，同时出现 3 这样一个操作 bar，可以停止和刷新已经在运行中的插件。还可以直接在对应的代码前加断点，执行命令触发断点后开始调试，也可以通过 console.log() 将日志打印在控制台，但是过于复杂的对象就不能显示了。

![image-20210415135934408](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f23ff9e41e844198b051cecbc117e97f~tplv-k3u1fbpfcp-zoom-1.image)

![image-20210415143508117](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0050024f24474e2aa43e44ce7d930d72~tplv-k3u1fbpfcp-zoom-1.image)

## 打包发布

- 安装 vsce(Visual Studio Code Extension)

  ```
  npm i vsce -g
  ```

- 打包成 vsix 文件：

  ```
  vsce package
  ```

  tips：生成的 vsix 文件不能直接拖入安装，只能从扩展的右上角选择 Install from VSIX 安装。

- 发布到应用市场

  注册登陆网站：登陆网站 https://dev.azure.com/vscode 获取一个 access token，用这个 token 来创建一个 publisher。

  创建发布者账号： vsce create-publisher (publisher name)

  登陆账号： vsce login (publisher name)

- 发布

  ```
  vsce publish
  ```

  增量发布：版本号(major.minor.patch)

  ```
  vsce publish patch(minor/major)
  ```

  取消发布：

  ```
  vsce unpublish (publisher name).(extension name)
  ```

- 更新

  修改版本号，重新执行 vsce publish 即可

## 问题及解决方案

1. 生成的 demo 运行后，触发 sayHello 的命令报错： ![image-20210330094850353](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6f82fc5636a431aa8a2b564b8adffdb~tplv-k3u1fbpfcp-zoom-1.image)

   原因是：vscode 版本低于插件要求的最低版本，即在 package.json 中定义的 engines 属性。

2. 写snippets时，测试发现并不生效，跟官方文档对比后，发现少写了一个属性 scope，之前看的教程比较老，没有这个属性也可以，但是看最新文档写的是四个属性都为必填值时才能生效。

   ![image-20210330173246357](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a6e948336664311865fb7d2521cdce0~tplv-k3u1fbpfcp-zoom-1.image)


