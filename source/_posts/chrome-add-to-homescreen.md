title: 在Android Chrome以app形式打开网站
date: 2015-05-11 10:34:14
categories: [网站]
tags: [Android, Chrome]
---
前面一篇文章我讲到了在UC网站可以以app形式打开，其实在Android Chrome浏览器上也支持此功能。
<!--more-->

## 添加配置文件
在网站根目录添加`manifest.json`，并进行相应配置
```json
{
  "name": "iblogc",
  "icons": [
    {
      "src": "launcher-icon-0-75x.png",
      "sizes": "36x36",
      "type": "image/png",
      "density": "0.75"
    },
    {
      "src": "launcher-icon-1x.png",
      "sizes": "48x48",
      "type": "image/png",
      "density": "1.0"
    },
    {
      "src": "launcher-icon-1-5x.png",
      "sizes": "72x72",
      "type": "image/png",
      "density": "1.5"
    },
    {
      "src": "launcher-icon-2x.png",
      "sizes": "96x96",
      "type": "image/png",
      "density": "2.0"
    },
    {
      "src": "launcher-icon-3x.png",
      "sizes": "144x144",
      "type": "image/png",
      "density": "3.0"
    },
    {
      "src": "launcher-icon-4x.png",
      "sizes": "192x192",
      "type": "image/png",
      "density": "4.0"
    }
  ],
  "start_url": "index.html",
  "display": "standalone",
  "orientation": "portrait"
}

```

## 在网站公用头部引入配置文件
```
<link rel="manifest" href="manifest.json">
```
## 查看效果
在Android使用Chrome打开网站，点击memu，选择“添加到主屏幕”选项，点击就可以添加到主屏幕了，步骤及显示效果截图如下：
![chrome-add-to-homescreen-01](/media/chrome-add-to-homescreen-01.png)
![chrome-add-to-homescreen-02](/media/chrome-add-to-homescreen-02.png)
PS:地址栏是不是不见了,看着像app而不是网页
![chrome-add-to-homescreen-03](/media/chrome-add-to-homescreen-03.png)
![chrome-add-to-homescreen-04](/media/chrome-add-to-homescreen-04.png)

## 扩展
ios的safari也有此功能，因手头无ios设备测试不了，所以内容不写了，大家可以参考此文章[http://www.prower.cn/technic/2314](http://www.prower.cn/technic/2314)

## 参考资料
> [https://developer.chrome.com/multidevice/android/installtohomescreen](https://developer.chrome.com/multidevice/android/installtohomescreen)


