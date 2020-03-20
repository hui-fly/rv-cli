相信你一定用过vue-cli或者create-react-app或者你公司自己的脚手架。当我第一次用这些脚手架的时候，会觉得这一定是种很高级的玩意，其实了解过之后就发现，脚手架也并不是多么深奥的东西。怎么做一个脚手架呢？接下来我会一步步的教你如何完成一个自己的脚手架（源码 https://github.com/hui-fly/rv-cli 记得star哦）

**原文地址：**   [一步步教你完成一个自己的脚手架并且发布到npm](https://www.yuque.com/no-bug/blog/xcglks)
   
## 1. 预备知识
本次教大家实现一个基础的cli工具，将会用到以下一些依赖，下面对这些依赖进行简单介绍：
### Inquirer.js
Inquirer.js试图为NodeJs做一个可嵌入式的美观的命令行界面。它的功能主要是：
- 询问操作者问题
- 获取并解析用户输入
- 检测用户回答是否合法
- 管理多层级的提示
- 提供错误回调
用法示例：
建立index.js，内容如下
```js
let inquirer = require('inquirer')
inquirer.prompt([
  {
    type: 'confirm',
    name: 'handsome',
    message: '我是世界上最帅的男人吗?',
    default: true
  }
]).then((answers) => {
  console.log(answers)
})
```
在文件目录线执行 node index.js 得到结果如下：
```{ handsome:true }```
这里type的类型包括
- input–输入
- validate–验证
- list–列表选项
- confirm–提示
- checkbox–复选框等等
想了解更多请参考：https://juejin.im/entry/5937c73cac502e0068cf1171

### commander
官方解释：commander灵感来自 Ruby，它提供了用户命令行输入和参数解析的强大功能，可以帮助我们简化命令行开发，它提供的功能是：
- 参数解析
- 强制多态
- 可变参数
- Git 风格的子命令
- 自动化帮助信息
- 自定义帮助等
说这么想必大家也比较抽象，直接上例子：
```
const program = require("commander");
// 定义指令
program
  .version('0.0.1')
  .usage('<command> [options]')
  .command('init', 'Generate a new project from a template')
  .action((option) => {
    // 回调函数
    console.log('Hello World')
  })
// 解析命令行参数
program.parse(process.argv);
```
同样在文件目录下执行 node command.js init，
然后就会发现输出  Hello World，
如果你想了解更多，可以参考 https://wangchujiang.com/wcj/#Commander

### download-git-repo
这个看名字大概就能猜出它是干嘛的了，没错---它就是帮我们完成下载远程仓库的，它的用法也比较简单，示例如下：
```
const download = require('download-git-repo')
download(repository, destination, options, callback)
```
其中 repository 是远程仓库地址；destination是存放项目的文件夹，下载完之后会默认建立在本目录下；options 是一些选项，比如 ```{ clone：boolean }``` 表示用 http download 还是 git clone 的形式下载，callback就是下载完成之后的回调函数了。
### chalk
这个依赖可以让你输出的内容变得更好看
```
const chalk = require('chalk');
console.log(chalk.green('success'));
console.log(chalk.red('error'));
```
### ora
这个依赖可以产生一个loading的效果，在进行下载的时候我们会用到它
OK，以上就是我们这次会用到的所有依赖了，接下里开始搞起我们的脚手架吧！
## 2. 脚手架开发
### 项目文件结构搭建
1. 首先初始化我们的项目，建立一个文件夹，我这里就叫rv-cli，你也可以随意取.
2. 建立好之后，在文件夹下执行npm             init（相信你一定已经安装了node了），一路回车即可，然后就会帮你生成一个package.json文件，在package.json添加如下代码--即我们预备知识介绍的依赖
    ```
    "dependencies": {
        "chalk": "^2.4.2",
        "commander": "^2.19.0",
        "download-git-repo": "^1.1.0",
        "inquirer": "^6.2.2",
        "ora": "^3.2.0"
    }
    ```
    然后执行npm i 安装这些依赖

3. 建立bin文件夹，并在bin文件夹下建立以下文件，这些文件就是我们执行的脚本，其    中rv-add用于添加项目，rv-delete用于删除一个项目，rv-init用于初始化一个项目    ，rv-list用于展示已添加的项目列表，而rv可以作为我们的入口文件。接下来我们    一步步完善这些脚本
   ![](https://user-gold-cdn.xitu.io/2020/3/20/170f6d2f576d5cae?w=398&h=322&f=png&s=23972)
4. 在建立的rv文件中添加一段熟悉的代码
    ```
    #!/usr/bin/env node
    console.log('hello world');
    ```
    然后执行 node ./bin/rv 就会输出 hello world了，这里大家会发现头部有一行#!/usr/bin/env node，这个的      作用就是告诉系统脚本的执行程序是node，并且系统会自动寻找node的所在路径。有了这行代码我们可以直      接执行 ./bin/rv而不用加node了

5. 我们仿照大多数的脚手架的设计，实现以下需求：在命令行输入rv add         即执行rv-add，执行rv delete即执行rv-delete... 其他同理，以下以rv-add为例。
首先我们先搞一下rv这个文件，话不多讲，直接贴代码：
    ```
    #!/usr/bin/env node
    const program = require('commander')
    // 定义当前版本
    // 定义使用方法
    // 定义四个指令
    program
      .version(require('../package').version)
      .usage('<command> [options]')
      .command('add', 'add a new template')
      .command('delete', 'delete a template')
      .command('list', 'list all the templates')
      .command('init', 'generate a new project from a template')
      
    // 解析命令行参数
    program.parse(process.argv)
    ```
    这里简单解释下：前面在定义的时候
    ```program.command('init').action(() => {})```其实本来还有个 action 回调，但是我们没写，commander 就会尝试在入口脚本的目录中搜索可执行文件，找到形如 program-command，于是这样的话，我们输入./bin rv add 执行的其实就是./bin rv-add
    
    然后再简单搞一下rv-add文件
    ```
    #!/usr/bin/env node
    console.log('hello rv-add!')
    ```
    当我们执行./bin/rv时可以看到输出了以下内容
    ![](https://user-gold-cdn.xitu.io/2020/3/20/170f6d3582fcba44?w=892&h=382&f=png&s=104282)
    接下来验证一下执行 ./bin rv add 是否执行了rv-add，在命令行输入./bin rv add，如果输出 hello rv-add，说明我们成功了！
    
    ### 完善各个脚本文件
    #### rv-add
    
    接下来我们完善一下rv-add，执行它之后帮我们添加一个项目，在完善它之前，我们首先在根目录下建立一个template.json，内容就是一个空对象{}即可。接下来直接贴代码
    
    ```
    #!/usr/bin/env node
    // 交互式命令行
    const inquirer = require('inquirer')
    // 修改控制台字符串的样式
    const chalk = require('chalk')
    // node 内置文件模块
    const fs = require('fs')
    // 读取根目录下的 template.json
    const tplObj = require(`${__dirname}/../template.json`)
    // 自定义交互式命令行的问题及简单的校验
    let question = [
      {
        name: "name",
        type: 'input',
        message: "请输入模板名称",
        validate (val) {
          if (val === '') {
            return 'Name is required!'
          } else if (tplObj[val]) {
            return 'Template has already existed!'
          } else {
            return true
          }
        }
      },
      {
        name: "url",
        type: 'input',
        message: "请输入模板地址",
        validate (val) {
          if (val === '') return 'The url is required!'
          return true
        }
      }
    ]
    inquirer
      .prompt(question).then(answers => {
        // answers 就是用户输入的内容，是个对象
        let { name, url } = answers;
        // 过滤 unicode 字符
        tplObj[name] = url.replace(/[\u0000-\u0019]/g, '')
        // 把模板信息写入 template.json 文件中
        fs.writeFile(`${__dirname}/../template.json`, JSON.stringify(tplObj), 'utf-8', err => {
          if (err) console.log(err)
          console.log('\n')
          console.log(chalk.green('Added successfully!\n'))
          console.log(chalk.grey('The latest template list is: \n'))
          console.log(tplObj)
          console.log('\n')
        })
      })
    ```
    注释写的也比较清楚，我就不多解释啦
    执行以下看看效果：
    ![](https://user-gold-cdn.xitu.io/2020/3/20/170f6d3bdb90bcc4?w=806&h=392&f=png&s=92032)
    这里先说明一下模版地址不需要完整的，只需要贴上下边红框内的即可
    ![](https://user-gold-cdn.xitu.io/2020/3/20/170f6d3fd37679bc?w=692&h=316&f=png&s=32365)
    
    #### rv-delete文件
    删除一个项目
    ```
    #!/usr/bin/env node
    const inquirer = require('inquirer')
    const chalk = require('chalk')
    const fs = require('fs')
    const tplObj = require(`${__dirname}/../template`)
    let question = [
      {
        name: "name",
        message: "请输入要删除的模板名称",
        validate (val) {
          if (val === '') {
            return 'Name is required!'
          } else if (!tplObj[val]) {
            return 'Template does not exist!'
          } else  {
            return true
          }
        }
      }
    ]
    inquirer
      .prompt(question).then(answers => {
        let { name } = answers;
        delete tplObj[name]
        // 更新 template.json 文件
        fs.writeFile(`${__dirname}/../template.json`, JSON.stringify(tplObj), 'utf-8', err => {
          if (err) console.log(err)
          console.log('\n')
          console.log(chalk.green('Deleted successfully!\n'))
          console.log(chalk.grey('The latest template list is: \n'))
          console.log(tplObj)
          console.log('\n')
        })
      })
      ```
    接下来看看执行的效果
![](https://user-gold-cdn.xitu.io/2020/3/20/170f6d43c776548b?w=780&h=336&f=png&s=66912)
    delete完之后，template就是{}了，因为之后我们还要用test模版，这里我们再把它add回去
    #### rv-list
    这个就比较简单了，就是输出项目列表
    ```
    #!/usr/bin/env node
    const tplObj = require(`${__dirname}/../template`)
    console.log(tplObj)
    ```
    执行结果如下：
    ![](https://user-gold-cdn.xitu.io/2020/3/20/170f6d8070943a66?w=784&h=116&f=png&s=41663)
    #### rv-init
    接下来是 init命令，这个要稍微麻烦一点，不过还好，代码走起

    ```
    #!/usr/bin/env node
    const program = require('commander')
    const chalk = require('chalk')
    const ora = require('ora')
    const download = require('download-git-repo')
    const tplObj = require(`${__dirname}/../template`)
    program
      .usage('<template-name> [project-name]')
    program.parse(process.argv)
    // 当没有输入参数的时候给个提示
    if (program.args.length < 1) return program.help()
    // 好比 vue init webpack project-name 的命令一样，第一个参数是 webpack，第二个参数是 project-name
    let templateName = program.args[0]
    let projectName = program.args[1]
    // 小小校验一下参数
    if (!tplObj[templateName]) {
      console.log(chalk.red('\n Template does not exit! \n '))
      return
    }
    if (!projectName) {
      console.log(chalk.red('\n Project should not be empty! \n '))
      return
    }
    url = tplObj[templateName]
    console.log(chalk.white('\n Start generating... \n'))
    // 出现加载图标
    const spinner = ora("Downloading...");
    spinner.start();
    // 执行下载方法并传入参数
    download (
      url,
      projectName,
      err => {
        if (err) {
          spinner.fail();
          console.log(chalk.red(`Generation failed. ${err}`))
          return
        }
        // 结束加载图标
        spinner.succeed();
        console.log(chalk.green('\n Generation completed!'))
        console.log('\n To get started')
        console.log(`\n    cd ${projectName} \n`)
      }
    )
    ```
    
    这里便用到了下载远程仓库的依赖 download-git-repo，这里我仿照的是
    vue init webpack project-name，webpack就是模版名，project-name就是项目名，也是项目的根目录，结下来我们执行一下
    rv init test demo 看看发生了什么吧
    ![](https://user-gold-cdn.xitu.io/2020/3/20/170f6d4a70751e05?w=798&h=322&f=png&s=53007)
执行之后，就开始下载远程仓库并生成demo文件夹，此时就生成了一个demo项目了
    ![](https://user-gold-cdn.xitu.io/2020/3/20/170f6d4f8f7dab19?w=418&h=480&f=png&s=64429)
但是大家有没有发现，我的init命令是 rv init test demo 而不是 ./bin/rv init test demo，如果你直接执行
 rv init test demo一定会失败，因为我做了一个操作让 rv 指向了./bin/rv，其实很简单，就是配置一下            package.json，即配置一下bin字段
     ```
    "bin": {
        "rv": "./bin/rv",
        "rv-add": "bin/rv-add",
        "rv-delete": "bin/rv-delete",
        "rv-list": "bin/rv-list",
        "rv-init": "bin/rv-init"
     },
     ```
    然后在package.json所在目录下执行npm link，这样便完成了命令的指向而且是全局的
    至此，我们的脚手架就基本完成了，给自己鼓个掌吧！！
## 3. 将脚手架发布到npm
1. 要发布npm包，首先你得有一个npm账号，注册完成之后，执行npm login命令输入你的用户名、密码、邮箱，然后就可以执行npm publish发布你的npm包了。但是rv-cli这个名字已经存在了，在发布之前，你得换个名字了，然后在npm上搜一下验证下你起的名字是不是已经被占用。哦对了，你还得把demo文件夹删掉，这个我们用来测试的就不要发上去了。
    ![](https://user-gold-cdn.xitu.io/2020/3/20/170f6d9b20772ed3?w=1034&h=626&f=png&s=265407)
2. 发布成功之后，你可以去npm上搜一下，就可以看到自己的包啦，鼓掌！！！
3. 最后我们在验证一波，是不是可以安装下来，命令行执行 npm i rv-cli -g，安装完之后，执行一下几个命令，耶，成功啦！！
    ![](https://user-gold-cdn.xitu.io/2020/3/20/170f6da0211a046a?w=1654&h=618&f=png&s=46737)

## 4. 总结
看到这里，你是不是也觉得一个脚手架也不过如此而已嘛，虽然这个还比较初级，但是其实也具备了基本的功能，好啦，我要去研究研究vue-cli了，等我研究完再来更新一篇

码字不易，欢迎点赞

参考: 

https://juejin.im/post/5a31d210f265da431a43330e

https://juejin.im/post/5c94fef7f265da60fd0c15e8