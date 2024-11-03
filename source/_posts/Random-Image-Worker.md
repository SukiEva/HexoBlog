---
title: CloudFlare Worker搭建随机图片API
toc: true
categories: Mixed
tags: CloudFlare Worker
date: 2022-06-17 12:30
updated: 2022-06-17 12:30
---

之前在 CSDN 上发表过 [制作自己的随机图API，快速、稳定、简易！](https://blog.csdn.net/qq_43640009/article/details/107945584)，主要通过一个简单的 php 脚本部署在服务器上。

这次通过 CloudFlare Worker 实现一个完全无成本的随机图片 API！

<!-- more -->

## 图片转 Webp

这步骤可选，转换为 webp 格式后，图片体积压缩而分辨率不变，可以更快地加载。

为了在引用时方便，先把名称按 1、2、3…顺序标号，再通过 `PIL` 库，将 `jpg` 和 `png` 格式的图片转换成 `webp`。

> 因为用了文件名按数字排序，所以原始图片需要全为数字，如为其他自行修改代码

```python
import os

from PIL import Image


def to_webp(src_dir, dist_dir, img):
    img_path = os.path.join(src_dir, img)
    dist_path = os.path.join(dist_dir, img.split(".")[0] + ".webp")
    if os.path.exists(dist_path):
        return
    image = Image.open(img_path)
    image.save(dist_path, format="webp")
    print(img_path + ' convert to ' + dist_path)


def convert(src_dir, dist_dir):
    print("Convert: start")
    images = os.listdir(src_dir)
    images.sort(key=lambda x: int(x[:-4]))
    for img in images:
        to_webp(src_dir, dist_dir, img)
    print("Convert: end")


def rename(dir_path):
    print("Rename: start")
    num = 1
    files = os.listdir(dir_path)
    files.sort(key=lambda x: int(x[:-4]))
    for file in files:
        src_path = os.path.join(dir_path, file)
        dist_path = os.path.join(dir_path, str(num)) + os.path.splitext(file)[-1]
        num = num + 1
        if src_path == dist_path:
            # print("Skip: " + src_path)
            continue
        os.rename(src_path, dist_path)
        print(src_path + ' rename to ' + dist_path)
    print("Rename: end")


if __name__ == '__main__':
    srcDir = r"E:\Onedrive\图片\次元壁纸"
    distDir = 'pc'

    rename(srcDir)
    convert(srcDir, distDir)
```

## 文件上传 Github

这步骤不具体说了，图片通过 `jsdeliver` 引用。

## CloudFlare Worker

新建一个 worker，代码参考如下：

- 因为改了文件名，所以下面只需要修改图片总数 `total`，就能生成一个随机数访问
- 个人通过 `type` 区别宽屏和竖屏图片，自己按需修改

```js
addEventListener('fetch', event => {
    event.respondWith(
        handleRequest(event.request).catch((err) =>
            new Response('cfworker error:\n' + err.stack, {
                status: 502,
            })
        )
    );
});

async function handleRequest(request) {
    const url = new URL(request.url);
    switch (url.pathname) {
        case "/img":
            var type = url.searchParams.has("type") ? url.searchParams.get("type") : "pc";
            const total = getTotal(type);
            if (total == 0) return handleImage("pc", getTotal("pc"));
            return handleImage(type, total);
        case "/favicon.ico":
            return fetch("https://cdn.jsdelivr.net/gh/SukiEva/assets/blog/favicon.ico");
        default:
            return handleNotFound();
    }
}

function getTotal(type) {
    switch (type) {
        case "pc": return 175;
        case "mb": return 0;
        default: return 0;
    }
}

async function handleImage(type, total) {
    var index = Math.floor((Math.random() * total)) + 1;
    var img = "https://cdn.jsdelivr.net/gh/SukiEva/assets/webp/" + type + "/" + index + ".webp";
    res = await fetch(img);
    return new Response(res.body, {
        headers: {
            'content-type': 'image/webp',
        },
    });
}

function handleNotFound() {
    return new Response('Not Found', {
        status: 404,
    });
}
```

## 自定义域名

默认是一个 CF 的域名，但是目前已经被国内 ban 了，需要自行添加域名路由。

- 添加 DNS A 解析到 `2.2.2.2`，我配置的 `api` 三级域名
- 在 Worker 里面配置路由，比如我的就是 `api.sukiu.top/img*`

可以通过下面的链接查看实现效果，每天的免费次数是 10w 次，请自行搭建，禁止滥用！！！

{% link SukiEva 的随机图片 API :: https://api.sukiu.top/img :: %}
