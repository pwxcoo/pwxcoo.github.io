---
title: 用node来发送邮件
date: 2018-05-12 21:08:41
categories: tech
tags: 
- javascript
- node
---

之前想做给 [pwxcoo.com](https://github.com/pwxcoo/BlackMagic) 做一个管理员的界面，看看网站日常是什么样的。初步是要做一个账号，然后管理员能看，然后巴拉巴拉，感觉有点麻烦，就一直搁置了。

然后看到一个 [nodemailer/nodemailer](https://github.com/nodemailer/nodemailer) 这个，用 node 来发邮件，感觉还行，就干脆直接用这个来把日志文件发给自己好了。

## 用 nodemailer 来发送邮件
直接用官网的 [demo](https://nodemailer.com/about/) 就好了。大概就是这样。

```JavaScript
const nodemailer = require('nodemailer');

let transporter = nodemailer.createTransport({
    // host: 'smtp.ethereal.email',
    service: 'qq', // 使用了内置传输发送邮件 查看支持列表：https://nodemailer.com/smtp/well-known/
    port: 465, // SMTP 端口
    secureConnection: true, // 使用了 SSL
    auth: {
        user: 'sender@qq.com',
        // 这里密码不是qq密码，是你设置的smtp授权码
        pass: '',
    }
});

let mailOptions = {
    from: '"sender" <sender@qq.com>', // sender address
    to: 'admin@qq.com', // list of receivers
    subject: 'Hello', // Subject line
    // 发送text或者html格式
    // text: 'Hello world?', // plain text body
    html: '<b>Hello world?</b>', // html body
    attachments: [{     // 附件
        // utf-8 string as an attachment
        filename: 'text1.txt',
        content: 'hello world!'
    },
    {   // file on disk as an attachment
        filename: 'text2.txt',
        path: '/path/to/file.txt' // stream this file

    }]
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

nodemailer 还是很强大的，可以用这个来给人发 html 邮件，很多公司的订阅都是这样实现的。

## 用 node-schedule 来定时发送
然后用一个 [node-schedule](https://github.com/node-schedule/node-schedule) 来做一个定时任务，定时发送邮件到管理者账号里来方便管理。

这个库的使用跟 linux 里的 cron 的语法是差不多的可以直接设置，demo 差不多是这样

``` JavaScript
var schedule = require('node-schedule');
 
var j = schedule.scheduleJob('26 * * * *', function(){
  console.log('I am running');
});

console.log('cron task is start!');
```

cron 的设置是否正确，可以用 [cron-parser](https://www.npmjs.com/package/cron-parser) 这个库来验证。下面这段代码将自动打印出下十次任务的执行时间，对照一下，就知道自己的 cron 设置是否正确了。

```JavaScript
const parser = require('cron-parser');
const interval = parser.parseExpression('0 59 23 * * *');

//打印后面10次的执行时间
for (let i = 0; i < 10; ++i) {
	console.log(interval.next().toString());
}
```

## 格式化时间
因为我想每天发一封，我的 log4js 里的设置就是每天写一个 log 文件的，文件的名字就是 `yyyy-mm-dd` 的设置，因为 log4js 把这个时间的模式匹配封装好了，所以我想用这个 pattern 来匹配日志文件然后发送，需要自己写一个格式化方法来解析。。

下面这个函数将返回 `yyyy-mm-dd` 格式的表示时间的字符串。
```JavaScript
function getNowFormatDate() {
    var date = new Date();
    var seperator1 = "-";
    var year = date.getFullYear();
    var month = date.getMonth() + 1;
    var strDate = date.getDate();
    if (month >= 1 && month <= 9) {
        month = "0" + month;
    }
    if (strDate >= 0 && strDate <= 9) {
        strDate = "0" + strDate;
    }
    var currentdate = year + seperator1 + month + seperator1 + strDate;
    return currentdate;
}
```

## 参考
- [将这个功能添加到我的node应用的具体实现](https://github.com/pwxcoo/BlackMagic/commit/171fcd4049aea05ca64d8db8b55665aaa637cbc2)
- [node-schedule/node-schedule](https://github.com/node-schedule/node-schedule)
- [nodemailer/nodemailer](https://github.com/nodemailer/nodemailer)
- [Node.js使用Nodemailer发送邮件](https://segmentfault.com/a/1190000012251328)