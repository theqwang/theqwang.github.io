title: 在django中把文件上传至七牛
date: 2015-12-22 12:51:42
categories: Python
tags: python
---
最近在写的一个django小项目需要实现用户上传图片的功能，使用到了七牛云存储，特此记录下来。这里我使用的七牛python SDK 版本是7.0.3，函数使用上可能会与旧版有些不同。

原本文件上传需要先把文件上传到自己的业务服务器，再从业务服务器上传到云存储。现在七牛的表单上传可以直接把文件上传到七牛，不再需要业务服务器的中转，节省了流量成本，降低了业务服务器的压力。而且通过设置，还可以在文件上传完成后让客户端自动重定向到一个上传成功的结果页面。这里我就是使用了七牛的表单上传。

<!--more-->

## 表单上传
用户上传图片的HTML表单代码如下。其中key用来指定图片保存在七牛中的文件名，token是上传凭证，即用来验证合法性和设置返回信息的。

```html
upload.html
<form method="POST" action="http://upload.qiniu.com/" enctype="multipart/form-data">
<input name="key" type="hidden" value="">
<input name="token" type="hidden" value="">
<input name="file" type="file">
<input  type="submit">
</form>
```

跳转到上面HTML页面的视图函数中的关键代码如下。其中upload_token函数用于生成表单里的token字段，upload_token函数中的7200代表上传凭证的有效期，returnUrl表示上传成功后的重定向地址，returnBody表示重定向时七牛返回的信息，它是一个base64编码后的json数据，需要解码获取json数据，当上传出错时错误信息直接在url中以明文的形式出现，并不会在返回的json数据里。通过设置mimeLimit还可以限制上传文件的类型。

```Python
views.py
import qiniu
import uuid
ACCESS_KEY = '七牛分配的公钥'
SECRET_KEY = '七牛分配的私钥'
BUCKET_NAME = '保存文件的仓库名'
key = str(uuid.uuid1()).replace('-', '')  # 这里使用uuid作为保存在七牛里文件的名字。并去掉了uuid中的“-”
q = qiniu.Auth(ACCESS_KEY, SECRET_KEY)
token = q.upload_token(BUCKET_NAME, key, 7200, {'returnUrl':'http://127.0.0.1:8000/photos/uploadprocessor', 'returnBody': '{"name": $(fname), "key": $(key)}', 'mimeLimit':'image/jpeg;image/png'})
return render_to_response('photos/upload.html', {'token': token, 'key': key}, context_instance=RequestContext(request))
```
