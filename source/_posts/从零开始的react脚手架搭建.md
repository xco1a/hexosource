---
title: 从零开始的react脚手架搭建
date: 2019-06-18 11:12:24
tags:
- 学习笔记
- React
categories: 
- 前端
---

一开始想要一个足够简洁，打包出的文件简单的脚手架，找了几天也没找到特别中意的，遂决定自己写一个。

为了让脚手架和项目模板分开管理、升级，因此创建了两个项目。cola-cli用于生成项目和一些node命令，cola-template用于存放项目模板。



## cola-cli



主要实现两个目的：

1，从cola-template的地址把项目模板文件下载到本地

2，用node实现一些自动添加组件、页面的方法



### 步骤



#### 来个init

```
npm init
```



为了解析node命令。安装commander依赖。大概想实现三个命令，-v查看版本号，init拉取项目模板，add添加一个组件。



#### 声明命令

新建bin文件夹，存放命令，cola文件注册命令。



```
#!/usr/bin/env node
// bin/cola
const program = require('commander')

program
  .version(require('../package.json').version, '-v, --version')
  .usage('<command> [options]')
  .command('add', 'add a new template')
  .command('init', 'generate a new project from a template')

program.parse(process.argv)
```



#### 拉取模板

如何把cola-template的文件拉取到本地。。试了好几个库总是报错，后来采用的是git-clone，简单粗暴的还真好用。。chalk用于美化命令行文字颜色，ora实现一个loading的效果。



```
#!/usr/bin/env node
// bin/cola-init
const program = require('commander')
const clone = require('git-clone')
const chalk = require('chalk')
const ora = require('ora')
const fs = require('fs')

function deleteall (path) {
  var files = []
  if (fs.existsSync(path)) {
    files = fs.readdirSync(path)
    files.forEach(function (file, index) {
      var curPath = path + '/' + file
      if (fs.statSync(curPath).isDirectory()) { // recurse
        deleteall(curPath)
      } else { // delete file
        fs.unlinkSync(curPath)
      }
    })
    fs.rmdirSync(path)
  }
}

program
  .usage('<name>')
program.parse(process.argv)
let name = program.args[0]
if (!name) {
  console.log(chalk.red(`\n Project must have a name ! `))
  throw new Error('param missing')
}
console.log(chalk.white('\n Start generating... \n'))
const spinner = ora('Downloading... \n')
spinner.start()

clone('http://gitlab.alibaba-inc.com/wb-wh449050/cola-template.git', name, (err) => {
  if (err) {
    spinner.fail()
    console.log(chalk.red('Error:' + err))
  } else {
    spinner.succeed()
    console.log(chalk.green(`\n Init succeed!`))
    console.log(`\n Please cd ${name} \n`)
    deleteall(`./${name}/.git`) // 递归删掉git文件，隐藏作案细节
  }
})
```



效果。。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/232869/1560826024026-f5625a4b-5f83-4dcb-9660-e41dedcf5f4a.png)



#### 添加一个组件

在项目里添加一个组件需要新建一个文件夹和写一堆初始代码。本着能懒则懒的原则。。

inquirer是个交互式命令行工具，这里用来实现让用户选择生成一个函数组件或者是一个标准组件。



```
#!/usr/bin/env node
const inquirer = require('inquirer')
const chalk = require('chalk')
const ejs = require('ejs')
const fs = require('fs')
const ora = require('ora')
const path = require('path')

let question = [
  {
    name: 'type',
    message: 'Choose one type : ',
    type: 'list',
    choices: ['Function Component', 'React.Component'],
    default: 0
  }, {
    name: 'name',
    message: "Component's name : ",
    type: 'string'
  }
]

inquirer
  .prompt(question).then((answer) => {
    let { type, name } = answer
    let Upper = name.split('')[0].toUpperCase()
    let nameArr = name.split('')
    nameArr.splice(0, 1, Upper)
    let Name = nameArr.join('')
    const srcpath = path.join(process.cwd(), 'src/components')
    const indexSrc = srcpath + '/' + Name + '/index.js'
    const lessSrc = srcpath + '/' + name + '/index.less'
    const Src0 = path.join(__dirname, './function_component_template.ejs')
    const Src1 = path.join(__dirname, './class_component_template.ejs')
    let str0 = ejs.render(fs.readFileSync(Src0, 'utf-8'), { name: name, Name: Name })
    let str1 = ejs.render(fs.readFileSync(Src1, 'utf-8'), { name: name, Name: Name })

    if (type === 'Function Component') {
      let spinner = ora('Establishing... \n')
      spinner.start()
      fs.mkdir(srcpath + '/' + Name, (err) => {
        if (err) {
          spinner.fail()
          console.log(chalk.red(err))
        } else {
          fs.writeFile(indexSrc, str0, (err) => {
            if (err) {
              spinner.fail()
              console.log(chalk.red(err))
            } else {
              fs.writeFile(lessSrc, `.component-${name}{}`, (err) => {
                if (err) {
                  spinner.fail()
                  console.log(chalk.red(err))
                } else {
                  spinner.succeed()
                  console.log(chalk.green(`\n ${Name} Creation Successful ! \n `))
                }
              })
            }
          })
        }
      })
    }
    if (type === 'React.Component') {
      let spinner = ora('Establishing... \n')
      spinner.start()
      fs.mkdir(srcpath + '/' + Name, (err) => {
        if (err) {
          console.log(chalk.red(err))
        } else {
          fs.writeFile(indexSrc, str1, (err) => {
            if (err) {
              console.log(chalk.red(err))
            } else {
              fs.writeFile(lessSrc, `.component-${name}{}`, (err) => {
                if (err) {
                  console.log(chalk.red(err))
                } else {
                  spinner.succeed()
                  console.log(chalk.green(`\n ${Name} Creation Successful ! \n `))
                }
              })
            }
          })
        }
      })
    }
  })
```



效果。。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/232869/1560826314998-2b9114e0-6373-42a4-b268-662fa2c4c598.png)



### 发布

emmm现在我们得到了一个脚手架，自己在本地玩的不是很方(zhuang)便(bi)。还想发布成npm包。。

要想发布，得有自己的账号，npm login。

登陆以后整理下自己的package.json。



```
{
  "name": "@ali/cola-cli",
  "version": "1.0.0",
  "description": "react-cli",
  "main": "index.js",
  "bin": {
    "cola": "bin/cola",
    "cola-init": "bin/cola-init",
    "cola-add": "bin/cola-add"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "publishConfig": {
    "registry": "https://registry.npm.alibaba-inc.com"
  },
  "author": "xcola",
  "license": "ISC",
  "dependencies": {
    "chalk": "^2.4.2",
    "commander": "^2.19.0",
    "ejs": "^2.6.1",
    "git-clone": "^0.1.0",
    "handlebars": "^4.1.1",
    "inquirer": "^6.2.2",
    "log-symbols": "^2.2.0",
    "ora": "^3.2.0"
  },
  "devDependencies": {},
  "repository": {
    "type": "git",
    "url": "git@git.emmm.com:emmm/cola-cli.git"
  }
}
```



然后。。npm publish就可以了。



## cola-template

这个模板文件，本着一切从简的原则，从头自己搭的乞丐版框架，酷炫狂拽的功能好像都没有。很多想加的功能，有空继续折腾。。

### 依赖

react，antd组件库，sass/less预处理器，typescript，webpack-server，redux-saga，axios。实现温饱的功能。。

```
"devDependencies": {
    "@babel/core": "^7.3.4",
    "@babel/preset-env": "^7.3.4",
    "@babel/preset-react": "^7.0.0",
    "autoprefixer": "^9.5.0",
    "babel-loader": "^8.0.5",
    "babel-plugin-transform-decorators-legacy": "^1.3.5",
    "babel-preset-react-app": "^7.0.2",
    "css-loader": "^2.1.1",
    "less": "^3.9.0",
    "less-loader": "^4.1.0",
    "node-sass": "^4.12.0",
    "postcss-loader": "^3.0.0",
    "style-loader": "^0.23.1",
    "ts-loader": "^5.3.3",
    "webpack": "^4.29.6",
    "webpack-cli": "^3.3.0",
    "webpack-dev-server": "^3.2.1"
  },
  "dependencies": {
    "@antv/g2": "^3.5.3",
    "@babel/polyfill": "^7.2.5",
    "@babel/runtime": "^7.3.4",
    "antd": "^3.15.1",
    "axios": "^0.18.0",
    "babel-plugin-import": "^1.11.0",
    "cookie": "^0.3.1",
    "react": "^16.8.4",
    "react-dom": "^16.8.4",
    "react-redux": "^7.0.3",
    "redux-saga": "^1.0.2"
  }
```



### 结构

src放代码，static放静态资源。redux-saga的过程略显复杂。。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/232869/1560827213490-e27e17c6-346b-47d6-a39d-fbbeac51fc57.png)



redux-saga流程。。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/232869/1560827348549-05fa72e2-14bc-4d5b-b2f3-8c769e97497c.png)

