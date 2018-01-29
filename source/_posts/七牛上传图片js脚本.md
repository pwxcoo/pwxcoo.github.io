---
title: 七牛上传图片js脚本
date: 2017-12-29 18:22:00
categories: 奇巧淫技
---

之前写 makedown 的时候，用了七牛云的图床，用的时候就在七牛的控制台里传图片，之前对图床的使用并不频繁，所以这样非常反人类的传图片方式也是也没注意，最近对图床使用极其频繁，所以用了几次之后感觉想吃一个至珍全虾堡冷静一下。

然后去看了一下七牛的[开发者文档](https://developer.qiniu.com/kodo/sdk/1289/nodejs)，写了一个 node 的传图片脚本，来冷静一下自己。

文档是中文的，写的也算详细。写了一个很简单的脚本。因为想以后通用这个脚本，所以对官方推荐稍微做了一点修改。

我把配置全部放在一个 json 里，这样以后再用别的 Bucket 的也通用这个脚本，重新配置一个 config.json 就 ojbk 了。

```javascript
{
    "accessKey": "账号密钥",
    "secretKey": "密码密钥",
    "bucket": "存储空间",
    "returnLink": "外链默认域名"
}
```

读取config.json的脚本

```javascript
const fs = require('fs');

var config = {}
function loadConfig(filename) {
	try {
		var data = fs.readFileSync(filename, 'utf-8');
		config = JSON.parse(data);
		//console.log("Loaded config '" + filename + "'");
	} catch (e) {
		console.log(e);
	}
}
var configFilename = './config.json';
loadConfig(configFilename);

module.exports = config

```

上传图片的脚本。

```javascript
const qiniu = require("qiniu");
const path = require('path');
const qiniuConfig = require('./config.js');

// localFile.split(/[\\/]/);

function getToken(key) {
    var mac = new qiniu.auth.digest.Mac(qiniuConfig.accessKey, qiniuConfig.secretKey);
    var putPolicy = new qiniu.rs.PutPolicy({
        scope: qiniuConfig.bucket + ":" + key,
        returnBody: '{"key":"$(key)","hash":"$(etag)","fsize":$(fsize),"bucket":"$(bucket)","name":"$(x:name)"}'
      });
    return putPolicy.uploadToken(mac);
}

function upload(key){
    var config = new qiniu.conf.Config();
    // 空间对应的机房
    config.zone = qiniu.zone.Zone_z0;
    var localFile = process.argv[2].trim(); 
    var key = path.basename(localFile);
    if(typeof(localFile) == "undefined"){
        console.log('please select file!')
        return;
    }
    var formUploader = new qiniu.form_up.FormUploader(config);
    var putExtra = new qiniu.form_up.PutExtra();
    
    // 文件上传
    formUploader.putFile(getToken(key), key, localFile, putExtra, function(respErr,
      respBody, respInfo) {
      if (respErr) {
        throw respErr;
      }
    
      if (respInfo.statusCode == 200) {
        //console.log(respBody);
        console.log(qiniuConfig.returnLink + key);
      } else {
        console.log(respInfo.statusCode);
        console.log(respBody);
      }
    });
}

upload();

```

用了 path 的依赖是为了让这个脚本在 linux 和 window 下都能通用，如果用的不是华东区，对应的机房也要更改。

![七牛脚本测试](http://p1pws2aca.bkt.clouddn.com/qiniu.png)

调用脚本，图片只要拉到 shell 窗口里就好了。然后可以直接返回图片的外链，复制粘贴就能用了。为了方便，生成的外链是根据图片的名字定的，所以要注意图片文件的命名规范了。

比我之前打开网页端登录，进控制台，传图片爽了些许。七牛上还有一些别的很秀的api，我暂时用不到，用到的时候再说吧。