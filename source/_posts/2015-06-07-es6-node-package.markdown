---
layout: post
title: "使用 BabelJS 编写 nodejs package"
date: 2015-06-07 15:45
comments: true
categories: 
tags: 
  - babel
  - es6
  - npm
---

我们都喜欢 ES6 优美的语法，更清晰的语义表达，所以用 ES6 语法编写 npm package 将是一件很快乐的事情。

### 初始化 npm package
```bash 
mkdir es6_project
cd es6_project
npm init .
```

系统在出现一连串提示后

```text
name: (es6_project)
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
About to write to /Users/home/es6_project/package.json:

{
  "name": "es6_project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

将会在当前目录下，创建一个 package.json 恭喜你一个 node package 已经创建成功了

### 怎么组织代码
为了让 es6 的代码，可以运行在任何环境下(browser, nodejs, es5) 需要编译与输出到新的代码，我通常会使用一个 下面这样的目录结构，当然，你可以使用任何你喜欢的一种方式，这只是一种个人选择而以
```
> tree
.
├── lib
├── src
├── index.js
└── package.json

2 directories, 1 file
```
`lib` 存放编译后的代码  
`src` 源代码目录  

两者目录结构相同，以便于代码调用，这可以由 Babel 编译器自动完成的

我们可以在 [`prepublish script`](https://docs.npmjs.com/misc/scripts) 脚本中，编译我们的代码，也可以在 [.npmignore](https://docs.npmjs.com/misc/developers#keeping-files-out-of-your-package) 中保留 src 中的代码

### 安装 Babel 支持
```
> npm install babel --save-dev
```

安装到 `devDependencies` 能让这个 package 能脱离 babel 运行，如果你只希望下载编译后的代码，放到 `devDependencies` 能让 `npm install` 不下载 Babel 编译器代码，让你的 package 更容易部署。

### 编译代码
当用 babel 编译代码的时候，我们可以用两种方式, 一种是 npm install -g babel 的方式，另一种则是，上面我说的到方式，为会我要说到两种方式呢，因为，当你使用 global 安装方式时候，你的项目会依赖于一个外部的全局环境，如果你想这样，也没有问题，但是如果你在开发时，或与人协作开发时，你希望部署与安装代码，更简单更自动化一点，你可以使用我推荐这种方式去做，安装 Babel 编译代码，这样你只要在下载后当前代码之后，轻松的建入 `npm install` 就能把你的开发环境准备好

使用 babel 编译代码的方式有很多，官方上面也有明确的说明 [https://babeljs.io/docs/setup/](https://babeljs.io/docs/setup/) 和大量的使用方式，因为我在这里使用需要编译后的代码，所以，我使用了 Makefile 这种简单的编译方式，当然，你还可能使用更多的，其它方式如: webpack, grunt, gulp,...等等工具来做 build 的事情；（可以参见 Babel 的官方资料), 先看简单代码

```makefile
# Makefile
SRC = $(shell find src -name '*.js')
LIB = $(SRC:src/%.js=lib/%.js)

lib: $(LIB)
lib/%.js: src/%.js
	mkdir -p $(@D)
	node_modules/.bin/babel --stage 0 $< -o $@
```

_注 后面的 --stage 是我加入的参数，你可以不使用这样的配置_

这样，`make` 就能帮助你编译代码了，make 这样的环境，主要是在 Mac OSX/ Linux， Windows 下面可以使用 cygwin 来替代

### 安装你的依赖包
安装依赖包是非常简单的事情，需要注意的是依赖包分为 `dependencies` 与 `devDependencies`, 两种引入方式，这两种引入方式主要的作用前面我提过，就是当外部的包依赖你时或其他开发者引入你的包时，将不会下载你的 `devDependencies` 包，这样的作法好处之一，就是让你的包的下载量大小，大大的降低，不需要在引用你的包的同时，将你所需要的编译，准备环境都下载下来，实际上就是让你能区分开 production 环境与 development 环境。这是特别要注意的，所以你在使用安装依赖包的时候，一定要思考清楚这个依赖包是不是一定要在发布后使用。

1. npm install --save [something package]
2. npm install --save-dev  [something package]

比如: `npm install --save-dev babel`; 这样别人在引你的 package 时 
```bash
> npm install your_packge --save 
```
将不会下载安装 babel

你可以使用 上面的命令，安装各种依赖包, 然后在 你的 .js 文件中引入  
如:
```bash 
> npm install --save debug
```
在 index.js

```js
var log = require('debug')('Es6Project');
/* ... */
```

es6 语法

```js 
import debug from 'debug';
const log = debug('ES6Proejct');
```

### 开始写代码
打开你的编辑器， 创建一个文件 `src/index.js` 开始编写你的代码。
```js
import assert from 'assert';

export default class Es6Project {
	/* ... */
}

```

make 会编译你的代码到 `lib` 目录下

### 测试
#### 1. 使用 [mocha](http://mochajs.org)  
```bash
npm install --save-dev mocha
```

#### 2. 建立 test 文件夹
```bash
mkdir test
```
#### 3. 生成 test 文件 
```		
mocha init test
```

#### 4. 加入 babel 支持在 test 模式
我在 test 模式下，使用是 `babel/register` 来支持 es6 
```js
// test/bootstrap.js
require("babel/register")(options); // options 是自定义的选项
```
当然也可以使用 
```
mocha --compilers js:babel/register"
```
#### 5. npm test
add to package.json
```json
  "description": "ES6Proejct ",
  "main": "index.js",
  "scripts": {
    "test": "./node_modules/.bin/mocha -r test/bootstrap"
  }
```
#### 6. 运行 npm test

```bash
touch test/testSpec.js
vi test/testSpec.js
```
```js 
var assert = require('assert');
var Es6Project = require('src/index.js');

describe('test', function() {
	it ('success', function() {
		assert.ok(Es6Project);
	});
});

```

你可测试与的代码了, 只要简单的 `npm test`



### 一些小技巧
#### 1. 发布前编译
package.json 
```json

{
  "description": "ES6Proejct ",
  "main": "index.js",
  "scripts": {
    "prepublish": "make",
  },
}
```

发布
```bash
npm publish
```
#### 2. 持续集成
先安装 nodemon
```bash
npm install --save-dev nodemon
```

加入 script 到 package.json 
```json

{
  "description": "ES6Proejct ",
  "main": "index.js",
  "scripts": {
    "dev": "./node_modules/.bin/nodemon -e js -x npm test",
  },
}
```
运行
```bash
npm run dev
```

当然修改代码一保存时，系统会自动进行测试
## 总结
当我们使用 Babel 进行开发的时候，既可以使用 ES6 先进语法，又能同时保持运行在 nodejs 中的性能与，而且同样支持 `commonjs/amd/requirejs` 的模块的引入与导出这种向下兼容性，所以是一种非常值得推荐的开发工具，希望能够给你的带来一些帮助。
