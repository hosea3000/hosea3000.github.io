# 使用 Nodemailer 发送邮件


## Node.js 使用 Nodemailer 发送邮件

## Nodemailer 简介

Nodemailer 是一个简单易用的 Node.js 邮件发送组件

官网地址：[https://nodemailer.com](https://nodemailer.com/)

GitHub 地址：https://github.com/nodemailer/nodemailer

Nodemailer 的主要特点包括：

- 支持 Unicode 编码
- 支持 Window 系统环境
- 支持 HTML 内容和普通文本内容
- 支持附件(传送大附件)
- 支持 HTML 内容中嵌入图片
- 支持 SSL/STARTTLS 安全的邮件发送
- 支持内置的 transport 方法和其他插件实现的 transport 方法
- 支持自定义插件处理消息
- 支持 XOAUTH2 登录验证

## 安装使用

首先，我们肯定是要下载安装 **注意：Node.js v6+**

```
npm install nodemailer --save
```

```js
'use strict';

const nodemailer = require('nodemailer');

let transporter = nodemailer.createTransport({
  // host: 'smtp.ethereal.email',
  service: 'qq', // 使用了内置传输发送邮件 查看支持列表：https://nodemailer.com/smtp/well-known/
  port: 465, // SMTP 端口
  secureConnection: true, // 使用了 SSL
  auth: {
    user: 'xxxxxx@qq.com',
    // 这里密码不是qq密码，是你设置的smtp授权码
    pass: 'xxxxxx',
  },
});

let mailOptions = {
  from: '"JavaScript之禅" <xxxxx@qq.com>', // sender address
  to: 'xxxxxxxx@163.com', // list of receivers
  subject: 'Hello', // Subject line
  // 发送text或者html格式
  // text: 'Hello world?', // plain text body
  html: '<b>Hello world?</b>', // html body
};

// send mail with defined transport object
transporter.sendMail(mailOptions, (error, info) => {
  if (error) {
    return console.log(error);
  }
  console.log('Message sent: %s', info.messageId);
  // Message sent: <04ec7731-cc68-1ef6-303c-61b0f796b78f@qq.com>
});
```

