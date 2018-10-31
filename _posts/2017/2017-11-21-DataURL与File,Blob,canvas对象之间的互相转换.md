---
layout: mypost
title: DataURL与File,Blob,canvas对象之间的互相转换
categories: [javascript]
---

## canvas转换为dataURL (从canvas获取dataURL)
```js
var dataurl = canvas.toDataURL('image/png');
var dataurl2 = canvas.toDataURL('image/jpeg', 0.8);
```

## File对象转换为dataURL、Blob对象转换为dataURL
File对象也是一个Blob对象，二者的处理相同。
```js
function readBlobAsDataURL(blob, callback) {
    var a = new FileReader();
    a.onload = function(e) {callback(e.target.result);};
    a.readAsDataURL(blob);
}
//example:
readBlobAsDataURL(blob, function (dataurl){
    console.log(dataurl);
});
readBlobAsDataURL(file, function (dataurl){
    console.log(dataurl);
});
```

## dataURL转换为Blob对象、dataURL转换为File对象
File继承于Blob，扩展了一些属性（文件名、修改时间、路径等）。绝大多数场景下，使用Blob对象就可以了。

兼容性：Edge浏览器不支持File对象构造函数，也就是Edge里不能new File()。
```js
function dataURLtoBlob(dataurl) {
    var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
    while(n--){
        u8arr[n] = bstr.charCodeAt(n);
    }
    return new Blob([u8arr], {type:mime});
}
function dataURLtoFile(dataurl, filename) {
    var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
    while(n--){
        u8arr[n] = bstr.charCodeAt(n);
    }
    return new File([u8arr], filename, {type:mime});
}
//test:
var blob = dataURLtoBlob('data:text/plain;base64,YWFhYWFhYQ==');
var file = dataURLtoFile('data:text/plain;base64,YWFhYWFhYQ==', 'test.txt');
```

## dataURL图片数据绘制到canvas
先构造Image对象，src为dataURL，图片onload之后绘制到canvas
```js
var img = new Image();
img.onload = function(){
    canvas.drawImage(img);
};
img.src = dataurl;
```

## File,Blob的图片文件数据绘制到canvas
还是先转换成一个url，然后构造Image对象，src为dataURL，图片onload之后绘制到canvas

利用上面的 readBlobAsDataURL 函数，由File,Blob对象得到dataURL格式的url，再参考 dataURL图片数据绘制到canvas
```js
readBlobAsDataURL(file, function (dataurl){
    var img = new Image();
    img.onload = function(){
        canvas.drawImage(img);
    };
    img.src = dataurl;
});
```
不同的方法用于构造不同类型的url (分别是 dataURL, objectURL(blobURL), filesystemURL)。这里不一一介绍，仅以dataURL为例。

filesystemURL不是指本地文件URL的形式(file:///….), 而是格式类似于 filesystem:http://... 的一种URL，支持沙盒文件系统的浏览器支持(目前仅Chrome)支持。

## Canvas转换为Blob对象并使用Ajax发送
转换为Blob对象后，可以使用Ajax上传图像文件。

先从canvas获取dataurl, 再将dataurl转换为Blob对象
```js
var dataurl = canvas.toDataURL('image/png');
var blob = dataURLtoBlob(dataurl);
//使用ajax发送
var fd = new FormData();
fd.append("image", blob, "image.png");
var xhr = new XMLHttpRequest();
xhr.open('POST', '/server', true);
xhr.send(fd);
```

[文章来源](https://blog.csdn.net/cuixiping/article/details/45932793)
