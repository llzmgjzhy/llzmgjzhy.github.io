---
layout: single
author_profile: true
read_time: true
title: 飞书群聊机器人每日自动推送语录及风景照片python版
share: true
---

> 飞书群聊机器人每日自动推送语录及风景照片python版

## 使用场景

最近使用飞书聊天，觉得蛮好用的，刚好这段时间在备考，就想着每天推送一些励志语录 和风景图片什么的，提一下动力

## 推送机制

飞书群机器人会有对应的webhook，顾名思义就是网络钩子，通过对钩子发送网络请求，即可将消息推送至群内，本文使用的python完成

## 代码目录

> #获取有效token以便进行图片上传
> def getttoken():
> 
> #在飞书中使用图片需要先将图片上传到飞书中，随后得到Image_Key便可发送图片
> def uploadImage():
> 
> #因为我想每天都发送新的图片，所以每次都许愿去网络上获取新的照片
> def getpic():
> main()

### 获取图片

在网上找到了一个图片API，更新序号即可获得不通过的图片，我以日期作为参考，每天获取一张新的照片，代码如下

```
def getpic():
    current_time = time.strftime('%Y-%m-%d', time.localtime(time.time()))
    start_date = '2022-2-17'
    end_date = current_time
    start_sec = time.mktime(time.strptime(start_date, '%Y-%m-%d'))
    end_sec = time.mktime(time.strptime(end_date, '%Y-%m-%d'))
    distance_days = int((end_sec - start_sec)/(24*60*60))
    days = 121+distance_days
    days = str(days)
    url = "https://cdn.jsdelivr.net/gh/flipped-1121/APIPIC@master/scenery/"+days+".jpg"
    name = "1.jpg"
    # 保存文件时候注意类型要匹配，如要保存的图片为jpg，则打开的文件的名称必须是jpg格式，否则会产生无效图片
    conn = urllib.request.urlopen(url)
    f = open(name, 'wb')
    f.write(conn.read())
    f.close()
    print('Pic Saved!')

```

### 获取有效的token

因为飞书中使用图片需要先将图片上传至飞书中，上传过程中需要获取tenant_access_token(企业自建应用)

图片功能需要机器人获得上传图片的权限，具体步骤[点击这里](图片功能需要机器人获得上传图片的权限，具体步骤点击这里)

代码如下
```
def gettoken():
    url='https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal?app_id=YOUR_ID&app_secret=YOUR_KEY'
    res = requests.get(url=url)
    token=res.json()
    token_text=token['tenant_access_token']
    return token_text

```

### 上传图片并得到image_key

上传图片文档请参考飞书[官方文档](https://link.csdn.net/?target=https%3A%2F%2Fopen.feishu.cn%2Fdocument%2FukTMukTMukTM%2FuEDO04SM4QjLxgDN)

代码如下

```
def uploadImage(token):
    url = "https://open.feishu.cn/open-apis/im/v1/images"
    form = {'image_type': 'message',
            'image': (open('./1.jpg', 'rb'))}  # 需要替换具体的path
    multi_form = MultipartEncoder(form)
    headers = {
        # 获取tenant_access_token, 需要替换为实际的token
        'Authorization': 'Bearer '+token,
    }
    headers['Content-Type'] = multi_form.content_type
    response = requests.request("POST", url, headers=headers, data=multi_form)
    # print(response.headers['X-Tt-Logid'])  # for debug or oncall
    res=bytes.decode(response.content)
    res=eval(res)
    print(res['data']['image_key']) # Print Response
    return res['data']['image_key']

```

## 使用

上传代码至服务器，调用.py文件即可