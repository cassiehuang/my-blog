---
title: "前端压缩之JSZip"
date: 2021-07-02T09:34:27+08:00
draft: false
---

这篇文章起源于一个工作中遇到的需求，虽然后面因为下载文件的总体积过大而没有选择这个方案，但是这个内容还是非常值得分享一下的。值得一提的是，JSZip的原理是将文件都下载到内存中，然后调用JSZip统一打包，然后触发保存，所以总的文件越大，性能越差，经过测试，1G以下的文件对性能影响较小，几个G的也能够处理，只是比较缓慢。所以，选用这个方案，建议总文件大小小于1G。
对于此需求，设计了如下demo：
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="./js/jszip.js"></script>
  <script src="./js/FileSaver.js"></script>
  <title>Document</title>
</head>
<body>
  <button id="download">下载并压缩</button>
</body>
<script>
  window.onload = function() {
    const fileAjax = (url, callback) => {
      console.log(url)
      const xhr = new XMLHttpRequest()
      xhr.open('get', url, true)
      xhr.responseType = 'blob'
      xhr.onreadystatechange = function () {
        if (xhr.readyState === 4 && xhr.status === 200) {
          callback(xhr.response)
        }
      }
      xhr.send()
    }
    
    const download = (urls) => {
      const zip = new JSZip()
      const folder = zip.folder('personInfomation')
      let curIndex = 0

      const downCallback = () => {
        fileAjax(urls[curIndex].url, (blob) => {
          folder.file(urls[curIndex].filename, blob)
          curIndex +=1
          if (curIndex < urls.length - 1) {
            downCallback()
          } else {
            zip.generateAsync({ type: 'blob'})
              .then((content) => {
                saveAs(content, 'test.zip')
              })
          }
        })
      }
      downCallback()
    }
    const urls = [
      {
        filename: 'test1.png',
        url: 'https://www.test.com/1.png'
      },
      {
        filename: 'test2.png',
        url: 'https://www.test.com/2.png'
      }
    ]

    const btn = document.querySelector('#download')
    btn.addEventListener('click', () => {
      download(urls)
    })
  }
</script>
</html>
```
实现的主要逻辑是，获取到文件名和下载地址后，循环的发起http请求，当一个请求完成后，将文件存放在zip文件内（此时zip还在内存区），然后再调起下一个http请求，知道文件全部下载完成。最后将内存区的zip文件保存到本地。
当然，要让这个方案更加完善，还可以使用promise和axios模拟多线程并发请求，同时考虑文件下载失败后重新发起请求等多种场景，这里为了讲目光集中在JSZip，就不再扩展开讲了。
JSZip的使用范围不止于此，它还可以运行在node环境，可以通过fileReader读取本地文件，然后进行压缩。因此，如果前端项目需要将文件打包为zip压缩包，可以在编译命令“npm run build”里面加上压缩的操作，不再使用手动压缩的方式。实现方式如下：
读取文件夹下的文件
如果是文件夹就用jszip创建文件夹，然后递归的执行readDir方法
如果是文件就读取文件，jszip创建文件
将内存区的jszip文件压缩
删除旧的zip包，将新的压缩包写入指定路径
```
const JSZip = require('jszip')
const fs = require('fs')
const path = require('path')

const readDir = (filePath, directory) => {
  const files = fs.readdirSync(filePath)
  files.forEach((fileName) => {
    const fileDir = path.join(filePath, fileName)
    const stats = fs.statSync(fileDir)
    if (stats.isFile()) {
      const content = fs.readFileSync(fileDir, 'utf8')
      directory.file(fileName, content)
    } else if (stats.isDirectory()) {
      const directory = jszip.folder(fileName)
      readDir(fileDir, directory)
    }
  })
}
const jszip = new JSZip()
const filePath = path.resolve('./cbkc-web')
readDir(filePath, jszip)

jszip
  .generateAsync({
    type: 'nodeBuffer',
    compression: 'DEFLATE',
    compressionOptions: {
      level: 9,
    },
  })
  .then((content) => {
    fs.unlinkSync('./cbkc-web.zip')
    fs.writeFileSync('./cbkc-web.zip', content)
  })
```
另一方面，JSZip还可以解压缩zip文件。在游戏开发中，尤其是小游戏平台，比如之前比较火热的h5小游戏，平台能够提供给游戏的包体积非常有限，这样的情况下，小游戏开发者不得不把资源压缩后存放在客户端，然后通过启动时读取文件后解压缩，这个时候就可以用到JSZip了。
