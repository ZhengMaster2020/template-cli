
# 根据需求搭建属于自己的脚手架

### 1. 处理终端用户输入的命令

1. 新建一个新文件夹，取名为`cm-cli`
2. 打开终端，输入命令`npm init -y`生成package.json文件
3. 安装相应的第三方库
`
 cnpm i commander download-git-repo inquirer  handlebars ora chalk log-symbols -S
`
+ **commander.js**，可以自动的解析命令和参数，用于处理用户输入的命令。
+ **download-git-repo**，下载并提取 git 仓库，用于下载项目模板。
+ **Inquirer.js**，通用的命令行用户界面集合，用于和用户进行交互。
+ **handlebars.js**，模板引擎，将用户提交的信息动态填充到文件中。
+ **ora，**下载过程久的话，可以用于显示下载中的动画效果。
+ **chalk**，可以给终端的字体加上颜色。
+ **log-symbols**，可以在终端上显示出 √ 或 × 等的图标。

4. 在cm-cli文件夹下新建一个`bin`文件夹，然后`bin`里面新建一个`project.js`文件

5. 在之前创建好的`package.json`文件里新增如下选项  
    ```json
      "bin": {
      "master-cli": "bin/project.js"
      },
    ```
    
    在`/cm-cli`目录下打开终端，使用`npm  link`或者`npm i . -g`进行全局安装我们自己的脚手架，这样我们就可以在终端任何地方使用`master-cli create <app-name>`进行脚手架的安装
    
6. 在`project.js`文件里面加入以下代码

   ```javascript
   #!/usr/bin/env node
    const program = require('commander')
    const chalk = require('chalk')
   
    program
    .version('1.0.0', '-V, --version')
    .usage('<command> [options]')
   
    program
      .command('create <app-name>')
      .description('create a new project powered by project-cli-service')
      .option('-d, --default', 'Skip prompts and use default preset')
      .action((name , cmd) => {
        console.log(chalk.blue(`${name}项目正在创建中...`))
   
    })
    program.parse(process.argv)
   ```

### 2. 从远程仓库下载模板

1. 前文已经安装了`download-git-repo`这个第三方库，接下来在`project.js`引入它

+ download-git-repo能够是我们从特定的远程仓库，GitHub，Gitlab等下载相应的项目模板  

+ Github官方网址：[https://github.com/flipxfx/download-git-repo#readme](https://github.com/flipxfx/download-git-repo#readme)    相关的使用方法进行自行浏览

+ ### 方法：download(repository, destination, options, callback)

  repository： 模板下载的仓库地址

  destination: 讲仓库的模板文件下载到本地哪里，

  options：其他选项 **clone： 使用git clone 命令下载模板** 

  All other options (`proxy`, `headers`, `filter`, etc.) will be passed down accordingly and may override defaults

  + 除clone的其他选项: https://github.com/kevva/download#options
  + clone的其他选项: https://github.com/jaz303/git-clone#clonerepo-targetpath-options-cb

```javascript
const download = reruire('download-git-repo')

const url = 'github.com:ZhengMaster2020/template-cli#master'
download(url, name, {clone:ture}, function (err) {
    if (err) {
     	console.log(chalk.red(err))   
     } else {
         // 进行package.json文件的内容替换操作
     }
})
```



###  3. 使用inquirer进行命令行交互

1. 在`project.js`引入inquirer第三方库

   ```javascript
   const inquirer = require('inquirer');
    inquirer
         .prompt([{
             name: 'version',
             message: '请输入项目版本',
             default: '1.0.0'
           },{
             name: 'description',
             message: '请输入项目描述信息',
             default: '这是一个自定义脚手架生成的项目'
           },{
             type: 'input'
             name: 'author',
             message: '请输入作者名称',
             default: 'zhengmaster'
           }])
   		.then(function (answer){
       	 	console.log(answer)
        		//进行下载模板
        		//模板的渲染等操作
   		 })
   ```

   这些内容是在中终端提示用户自行输入相应的内容来生成相应的**package.json**文件的

   prompt方法：接受一个数组，数组放多个对象，每个对象表示一个含义，name表示含义的名称，message表示含义的描述，default为描述提供了一个默认值，type表示类型，值为input表示输入类型。

   then方法：prompt的回调函数，prompt返回一个answer（就是用户在终端输入的内容）它接受一个answer作为参数
   更多使用方法的详情：[https://blog.csdn.net/qq_26733915/article/details/80461257](https://blog.csdn.net/qq_26733915/article/details/80461257)

###  4. 渲染模板

这里面要使用到的是模板引擎，有**handlebar**（本次我们使用的是这个），**ejs**, **jade** 等等

还有使用到了Node的fs模块：`const fs = require('fs')`

**当下载模板完成之后会将用户输入的答案渲染到 package.json 中**

我的模板仓库：[https://github.com/ZhengMaster2020/template-cli](https://github.com/ZhengMaster2020/template-cli)
模板项目里面的`package.json`已经修改成如下：

```json
{
  "name": "{{name}}", //使用用户输入的name内容替换掉
  "version": "{{version}}", //用用户填的version内容替换
  "description": "{{description}}",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/ZhengMaster2020/template-cli.git"
  },
  "keywords": [],
  "author": "{{author}}",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/ZhengMaster2020/template-cli/issues"
  },
  "homepage": "https://github.com/ZhengMaster2020/template-cli#readme"
}

```



### 5. 终端现实的美化

在用户输入答案之后，开始下载模板，这时候使用 ora 来提示用户正在下载中。

```javascript
const ora = require('ora');
// 开始下载
const spinner = ora('正在下载模板...');
spinner.start();

// 下载失败调用
spinner.fail();

// 下载成功调用
spinner.succeed();
```

然后通过 chalk 来为打印信息加上样式，比如成功信息为绿色，失败信息为红色，这样子会让用户更加容易分辨，同时也让终端的显示更加的好看。

```javascript
const chalk = require('chalk');
console.log(chalk.green('项目创建成功'));
console.log(chalk.red('项目创建失败'));
```

除了给打印信息加上颜色之外，还可以使用 log-symbols 在信息前面加上 √ 或 × 等的图标

```javascript
const chalk = require('chalk');
const symbols = require('log-symbols');
console.log(symbols.success, chalk.green('项目创建成功'));
console.log(symbols.error, chalk.red('项目创建失败'));
```

### 6. 完整内容

```javascript
#!/usr/bin/env node

const program = require('commander')
const chalk = require('chalk')
const download = require('download-git-repo')
const inquirer = require('inquirer')
const fs = require('fs')
const ora = require('ora')
const symbols = require('log-symbols')
const handlebar = require('handlebars')


program
  .version('1.0.0', '-V   --version')
  .usage('<command> [options]')


program
  .command('create <app-name>')
  .description('create a new project powered by project-cli-service')
  .option('-d, --default', 'Skip prompts and use default preset')
  .action((name) => {
    console.log(chalk.blue(`${name}项目正在创建中...`))
    if (!fs.existsSync(name)) {
      inquirer
      .prompt([{
          name: 'version',
          message: '请输入项目版本',
          default: '1.0.0'
        },{
          name: 'description',
          message: '请输入项目描述信息',
          default: '这是一个自定义脚手架生成的项目'
        },{
          name: 'author',
          message: '请输入作者名称',
          default: ''
        }])
      .then(answer => {
        const url ='github.com:ZhengMaster2020/template-cli#master'
        const spinner = ora(`正在下载模板，源地址：${url}`)
        spinner.start()
        download(url, name, {clone: true}, function (err) {
          if (err) {
            spinner.fail()
            console.log(symbols.error, chalk.red(err))
          } else {
            spinner.succeed()
            const fileName = name+'/package.json'
            const meta = {
              name,
              version: answer.version,
              description: answer.description,
              author: answer.author
            }
            // {{name}}/package.json路径存在时
            if (fs.existsSync(fileName)) {
              const content = fs.readFileSync(fileName).toString()
              const resultContent = handlebar.compile(content)(meta)
              fs.writeFileSync(fileName, resultContent)
            }
            console.log(symbols.success, chalk.green('项目初始化成功'))
            console.log(symbols.info, chalk.yellow('cd'+' '+ name))
            console.log(symbols.info, chalk.yellow('npm run serve'))
            console.log(symbols.info, chalk.yellow('npm run build'))
          }
        })
      })
    } else {
      console.log(symbols.error, chalk.red('项目已存在'))
    }
  })

program.parse(process.argv)
```

### 最终现实

执行`master-cli create my-cli`会在当前打开的终端目录下创建一个`my-cli`文件夹，这个文件夹就是我们的脚手架生成的项目。

```powershell
zhengmaster MINGW64 /f/Front-end/vue-project/cm-cli
$ master-cli create my-cli
my-cli项目正在创建中...
? 请输入项目版本 1.0.0
? 请输入项目描述信息 这是一个自定义脚手架生成的项目
? 请输入作者名称
√ 正在下载模板，源地址：github.com:ZhengMaster2020/template-cli#master
√ 项目初始化成功
i cd my-cli
i npm run serve
i npm run build
```

### 项中遇到的坑
```javascript
//在顶部添加这句:
#!/usr/bin/env node  --这种用法是为了防止操作系统用户没有将node装在默认的/usr/bin路径里。当系统看到这一行的时候，
首先会到env设置里查找node的安装路径，再调用对应路径下的解释器程序完成操作。
//download-git-repo踩坑(路径错误导致下载模板失败--git clone status 128)
//从github上下载所需得template 下载地址不是你复制得https://github.com/xxx/xxx.git
//正确写法：
download('github.com:ZhengMaster2020/template-cli#master', name, {clone: true}, (err) => {
    console.log(err ? 'Fail' : 'Success')
})
//还有一种简写：
ZhengMaster2020/express-tpl#template-cli#master
//#master为模板所在的分支
```

本脚手架只是简单完成了一个 create命令来创建一个项目，更多的功能日后更新。。。

---
我的GitHub：[https://github.com/ZhengMaster2020](https://github.com/ZhengMaster2020)
更多脚手架的内容请自行参考：

vue-cli官方网站：[https://cli.vuejs.org/zh/](https://cli.vuejs.org/zh/)

vue-cli源码与原理实现：[https://kuangpf.com/vue-cli-analysis/](https://kuangpf.com/vue-cli-analysis/)

