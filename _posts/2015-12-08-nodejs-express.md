---
layout: post
title: Mac上搭建nodejs开发环境
description: ""
modified: 2015-12-08
tags: [nodejs,express]
image:
  feature: abstract-6.jpg
  
---

###Mac上搭建nodejs开发环境
- - -

####安装homebrew

~~~
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew update
brew doctor
~~~

####安装nodejs

通过brew安装nodejs和grunt
~~~
brew install node
npm install -g grunt-cli
~~~

安装成功

~~~
node -v
v4.2.1
npm -v
2.14.7
~~~

####安装express

~~~
sudo npm install express -g
sudo npm install -g express-generator@4
/usr/local/bin/express -> /usr/local/lib/node_modules/express-generator/bin/express
express-generator@4.13.1 /usr/local/lib/node_modules/express-generator
├── sorted-object@1.0.0
├── commander@2.7.1 (graceful-readlink@1.0.1)
└── mkdirp@0.5.1 (minimist@0.0.8)
~~~

~~~
express -V
4.13.1
~~~

安装成功

####建立express工程

~~~
mkdir javascript
cd javascript

express -e nodejs-demo

   create : nodejs-demo
   create : nodejs-demo/package.json
   create : nodejs-demo/app.js
   create : nodejs-demo/public
   create : nodejs-demo/public/javascripts
   create : nodejs-demo/public/images
   create : nodejs-demo/public/stylesheets
   create : nodejs-demo/public/stylesheets/style.css
   create : nodejs-demo/routes
   create : nodejs-demo/routes/index.js
   create : nodejs-demo/routes/users.js
   create : nodejs-demo/views
   create : nodejs-demo/views/index.ejs
   create : nodejs-demo/views/error.ejs
   create : nodejs-demo/bin
   create : nodejs-demo/bin/www

   install dependencies:
     $ cd nodejs-demo && npm install

   run the app:
     $ DEBUG=nodejs-demo:* npm start~~~

为项目安装依赖包

~~~
d nodejs-demo && npm install
ejs@2.3.4 node_modules/ejs

debug@2.2.0 node_modules/debug
└── ms@0.7.1

cookie-parser@1.3.5 node_modules/cookie-parser
├── cookie@0.1.3
└── cookie-signature@1.0.6

serve-favicon@2.3.0 node_modules/serve-favicon
├── fresh@0.3.0
├── etag@1.7.0
├── ms@0.7.1
└── parseurl@1.3.0

morgan@1.6.1 node_modules/morgan
├── on-headers@1.0.1
├── basic-auth@1.0.3
├── depd@1.0.1
└── on-finished@2.3.0 (ee-first@1.1.1)

express@4.13.3 node_modules/express
├── escape-html@1.0.2
├── merge-descriptors@1.0.0
├── cookie@0.1.3
├── array-flatten@1.1.1
├── utils-merge@1.0.0
├── cookie-signature@1.0.6
├── methods@1.1.1
├── content-type@1.0.1
├── range-parser@1.0.3
├── fresh@0.3.0
├── etag@1.7.0
├── serve-static@1.10.0
├── vary@1.0.1
├── path-to-regexp@0.1.7
├── content-disposition@0.5.0
├── parseurl@1.3.0
├── depd@1.0.1
├── qs@4.0.0
├── on-finished@2.3.0 (ee-first@1.1.1)
├── finalhandler@0.4.0 (unpipe@1.0.0)
├── proxy-addr@1.0.9 (forwarded@0.1.0, ipaddr.js@1.0.4)
├── send@0.13.0 (destroy@1.0.3, statuses@1.2.1, ms@0.7.1, mime@1.3.4, http-errors@1.3.1)
├── type-is@1.6.10 (media-typer@0.3.0, mime-types@2.1.8)
└── accepts@1.2.13 (negotiator@0.5.3, mime-types@2.1.8)

body-parser@1.13.3 node_modules/body-parser
├── content-type@1.0.1
├── bytes@2.1.0
├── depd@1.0.1
├── qs@4.0.0
├── on-finished@2.3.0 (ee-first@1.1.1)
├── iconv-lite@0.4.11
├── http-errors@1.3.1 (statuses@1.2.1, inherits@2.0.1)
├── type-is@1.6.10 (media-typer@0.3.0, mime-types@2.1.8)
└── raw-body@2.1.5 (unpipe@1.0.0, bytes@2.2.0, iconv-lite@0.4.13)
~~~

启动应用

~~~
npm start

> nodejs-demo@0.0.0 start /Users/huangjie/javascript/nodejs-demo
> node ./bin/www
~~~

测试是否启动成功

~~~html
curl http://localhost:3000
<!DOCTYPE html>
<html>
  <head>
    <title>Express</title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
    <h1>Express</h1>
    <p>Welcome to Express</p>
  </body>
~~~

ok，至此，nodejs的express框架环境已经配置成功，接下来就可以享受nodejs开发的快感了....