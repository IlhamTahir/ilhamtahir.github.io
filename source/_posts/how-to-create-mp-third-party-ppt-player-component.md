---
title: 利用小程序第三方自定义组件能力开发文档播放器组件
date: 2021-01-06 00:06:35
tags:
---

# 利用小程序第三方自定义组件能力开发文档播放器组件

## 起因
前段时间，承担公司内部智慧课堂小程序项目的技术leader。客户方由于是出版社，对于图书资源播放有高度定制要求，鉴于公司本身文档播放器未能提供原生小程序SDK，我利用小程序第三方自定义组件能力去开发了一个简单的文档播放器组件。
<!-- more -->
## 准备工作

安装小程序第三方命令行工具, 官方提供了命令行工具，用于快速初始化一个项目。执行如下安装命令：

```
npm install -g @wechat-miniprogram/miniprogram-cli
```

创建一个空目录，在目录中执行初始化第三方自定义组件示例项目命令。

```
# 创建空目录
mkdir document-player-miniprogram

# 进入目录
cd document-player-miniprogram

# 初始化示例项目
miniprogram init --type custom-component
```

## 目录结构

以下为文档播放器目录结构:

```
|--miniprogram_dev // 开发环境构建目录
|--miniprogram_dist // 生产环境构建目录
|--src // 源码
|   |--assets // 静态资源
|   |--aes-decrypter.js // AES解密器
|   |--index(.wxml,.wxss,.ts,.json) // 播放器组件
|   |--scale-image(.wxml,.wxss,.ts,.json) // 单个图片缩放器组件
|   |--types.ts // 接口类
|--test // 测试用例
|--tools // 构建相关代码
|   |--demo // demo 小程序目录，开发环境下会被拷贝生成到 miniprogram_dev 目录中
|   |--config.js // 构建相关配置文件
|
|--gulpfile.js
```

## 开发

根据官方文档，安装依赖

```
npm install
```

启动监听

```
npm run watch
```

接下来我们就可以正式进入开发自己的自定义组件了。


## 相关特性

### 播放类型选择

根据传入参数imageData的长度是否存在来判断播放器加载图片的模式：

```
      if (this.properties.imageData.length) {
        this._initNoEncryptData()
      } else {
        this._initEncryptData()
      }
```

### 非加密播放

非加密播放时采用imageData数据构造imageList数据，并传入isEncrypted：false

```
      const list = []
      for (let i = 0; i < this.properties.imageData.length; i++) {
        const img = {
          src: this.properties.imageData[i],
          page: (i + 1),
          active: false,
          load: false
        } as ImgItem
        list.push(img)
      }
      this.setData({
        imageList: list,
        isEncrypted: false
      })
```

### 加密播放

加密播放时，imageUrlPrefix参数生效，并构造imageList，传入isEncrypted：true

```
      this.setData({
        imageList: list,
        isEncrypted: true
      })
```



### 加载图片

调用goto方法，传入加载page

```
      this.goto({page: 1})

```


根据load参数判断图片是否预加载完成

```
        if (!newActiveImg.load) {
          this._loadImage(newActiveImg)
        }
```


由于图片资源加载需要过程，前端未加载图片暂时有了一个placehold图片。

```
        <scale-image ... src="{{ item.load ? item.base64 : './assets/loading.gif' }}" ... ></scale-image>
```

根据图片资源先调用wx.request 方式预加载图片内容

```
      wx.request({
        url: image.src,
        responseType: 'arraybuffer',
        success: result => {
        ...
        }
```

加密模式，采用加密算法解密图片内容并转换base64

```
          if (that.data.isEncrypted) {
            const key = that._str2uint32(this.properties.encryptKey)
            const iv = that._str2uint32(this.properties.encryptIv)
            const readerBuffer = new Uint8Array(data)
            // eslint-disable-next-line handle-callback-err,no-new
            new Decrypter(readerBuffer, key, iv, (err, decryptedArray) => {
              image.base64 = 'data:image/png;base64,' + wx.arrayBufferToBase64(decryptedArray.buffer)
              image.load = true
              const currentIndex = image.page - 1
              that.setData({
                ['imageList[' + currentIndex + ']']: image,
              })
            })
          } 
```

非加密模式，内容函数直接转换base64

```
            image.base64 = 'data:image/png;base64,' + wx.arrayBufferToBase64(data)
            image.load = true
            const currentIndex = image.page - 1
            that.setData({
              ['imageList[' + currentIndex + ']']: image,
            })
```


## 发布组件

生成build内容

```
npm run build
```

设置版本号

```
package.json
{
  "name": "document-player-miniprogram",
  "version": "1.3.7", // 修改
  "description": "",
  "main": "miniprogram_dist/index.js",
```

执行发布脚本

```
npm run publish-to-npm
```


## 在小程序仓库中调用

升级安装npm包


```
npm install document-player-miniprogram --registry  https://registry.npmjs.org/

```

构建npm即可


