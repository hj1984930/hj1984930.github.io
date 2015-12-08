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
sudo npm install express@3.0.6 -g
/usr/local/bin/express -> /usr/local/lib/node_modules/express/bin/express
express@3.0.6 /usr/local/lib/node_modules/express
├── methods@0.0.1
├── fresh@0.1.0
├── cookie-signature@0.0.1
├── range-parser@0.0.4
├── buffer-crc32@0.1.1
├── cookie@0.0.5
├── commander@0.6.1
├── mkdirp@0.3.3
├── debug@2.2.0 (ms@0.7.1)
├── send@0.1.0 (mime@1.2.6)
└── connect@2.7.2 (pause@0.0.1, bytes@0.1.0, qs@0.5.1, formidable@1.0.11)
~~~

安装成功

####建立express工程

~~~
xpress -e nodejs-hj

   create : nodejs-hj
   create : nodejs-hj/package.json
   create : nodejs-hj/app.js
   create : nodejs-hj/public
   create : nodejs-hj/public/javascripts
   create : nodejs-hj/public/images
   create : nodejs-hj/public/stylesheets
   create : nodejs-hj/public/stylesheets/style.css
   create : nodejs-hj/routes
   create : nodejs-hj/routes/index.js
   create : nodejs-hj/routes/user.js
   create : nodejs-hj/views
   create : nodejs-hj/views/index.ejs

   install dependencies:
     $ cd nodejs-hj && npm install

   run the app:
     $ node app
~~~

为项目安装依赖包

~~~
cd nodejs-hj/
sudo npm install
ejs@2.3.4 node_modules/ejs

express@3.0.6 node_modules/express
├── methods@0.0.1
├── fresh@0.1.0
├── range-parser@0.0.4
├── cookie-signature@0.0.1
├── buffer-crc32@0.1.1
├── cookie@0.0.5
├── commander@0.6.1
├── mkdirp@0.3.3
├── send@0.1.0 (mime@1.2.6)
├── debug@2.2.0 (ms@0.7.1)
└── connect@2.7.2 (pause@0.0.1, bytes@0.1.0, qs@0.5.1, formidable@1.0.11)
~~~

启动应用

~~~
node app.js
Express server listening on port 3000
GET / 200 13ms - 206
~~~

测试是否启动成功

~~~curl http://localhost:3000
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