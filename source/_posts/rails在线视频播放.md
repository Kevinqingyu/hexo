---
title: rails在线视频播放
date: 2018-04-26 16:22:04
tags: [ruby, Dplayer, rails]
---

## rails网站实现在线视频播放
![](https://rails365.oss-cn-shenzhen.aliyuncs.com/uploads/photo/image/656/2018/2da9e44c6a2c6dd28a2d1fa6bc67a7b8.jpg)
<!--more-->
### 准备

>播放器使用 [DPlayer](http://dplayer.js.org/#/)一个好用的弹幕视频播放器。

>首先将js文件下载后引用 application.js中添加


```js
//= require DPlayer.min
```

> 页面中添加

```html
<div id="dplayer"></div>

<script>
  const dp = new DPlayer({
    container: document.getElementById('dplayer'),
    video: {
      url: "<%= @match.get_video_url.html_safe %>",
      pic: "<%= @match.get_pic_url.html_safe %>",
    },
  });
</script>
```

>url 是可播放的视频地址, pic 是视频的封面图片
如果相接弹幕也是可以的，可以点连接后仔细了解，非常好用的视频播放器。
