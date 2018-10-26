---
layout: mypost
title: apicloud开发混合app一期总结
categories: [apicloud]
---

本次开发使用[HUI](http://www.hcoder.net/hui/docs349.html)为UI库，然而**并不好用，不推荐使用**。尽量使用apicloud官方提供的功能。

## html头部设置
```html
<meta name="viewport" content="maximum-scale=1.0,minimum-scale=1.0,user-scalable=0,width=device-width,initial-scale=1.0"/>
```

## 字体、宽高等px、rem设置
传统web开发一般采用px为单位，H5开发app时推荐使用rem，根据body的font-size自动计算出对应数值。

下面代码可设置body的px。
```javascript
(function (doc, win) {
    var docEl = doc.documentElement,
        //如果有orientationchange事件则赋值该事件，否则赋值resize事件
        //orientationchange:屏幕翻转事件；resize:浏览器大小改变事件
        resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
        recalc = function () {
            var clientWidth = docEl.clientWidth;
            if (!clientWidth) return;
            docEl.style.fontSize = 100 * (clientWidth / 320) + 'px';
        };
    // Abort if browser does not support addEventListener
    if (!doc.addEventListener) return;
    //给win监听上面赋值的resizeEvt事件
    win.addEventListener(resizeEvt, recalc, false);
    //给doc监听DOMContentLoaded事件
    //jQ中$(document).ready() 监听的就是 DOMContentLoaded 事件，而 $(document).load()监听的是 load 事件
    doc.addEventListener('DOMContentLoaded', recalc, false);
})(document, window);
```
*上面代码中设置font-size的公式可根据设计稿改变，比如UI图是以750px宽度标准设计的，公式可写为100 \* (clientWidth / 750) + 'px'，这样在750px宽度浏览器下 body 的 fontSize 为100px。比如设计稿中某p元素的字体为30px，只要写为0.3rem即可，省去很多计算上的麻烦。这样在其他宽度下会适当放大缩小。*

*公式中有乘以100，是为了防止计算出来的fontSize太小，比如UI稿为750px，但实机宽度只有一半，除后只有0.5，乘以10为5，都小于12px，而chrome中小于12px的fontSize都会按12px来算，照成误差。*


## ajax
### 参数传递
apicloud提供了ajax接口，需要注意的是data中需要多设置一层body

### cookie传递
平台上ajax请求好像不会自动传递cookie，原因不明。

解决方法是在headers头设置Cookie，这里注意大小写，否则可能失效。另外后端传回的 cookie 无法获取，需要后端直接在返回参数中传递，再自己设置。不推荐设置在cookie中，可以设置在localstorage。
```javascript
api.ajax({
    url: 'url',
    method: 'post',
    data: {
        body: {
            //这里是传递的参数
        }
    },
    headers: {
        'Content-Type': 'application/json',
        'Cookie': 'key=value;'  //Cookie传递，注意大小写
    }
}, function(ret, err) {
    if (ret) {
        //success
    } else {
        //fail
    }
})
```

## apiready
若要使用平台提供的api.xx方法，需要满足如下条件
```JavaScript
apiready = function(){
    //这里面才能用api.xxx
}
```
因为在项目中用了vue，这里采用这种方法
```JavaScript
apiready = newVue；
function newVue(){
    new Vue({
        //...
    })
}
```
若使用两个 apiready = function(){} ，前面一个会被覆盖。有一种情况是进入页面后要在公共js中调用某些公共方法，且方法包含api.xxx，这样就不能直接在公共js中写自执行函数，而在公共js中写 apiready 也会被页面中的覆盖。

**解决**
```JavaScript
//public.js
function public(){
    //一些公共方法
}
//a.js (页面a的js文件)
apiready = function(){
    public();
    //其他方法
}
//虽然想过把apiready放在public.js中，然后里面调用 pageFn ，在各个页面的js定义 pageFn ，但这样public.js中没有pageFn方法就会报错。
```

## 其他常用方法
### $api
```javascript
$api.byId('a')  //根据id获取元素
$api.dom('.b'); //选择器，选择符合的第一个
$api.domAll('.b'); //选择器，选择全部符合的
//给dom设置padding-top,防止遮住手机上部分，一般设置header
$api.fixStatusBar(dom)
$api.val(dom[, val])    //获取或设置val
$api.text(dom[, val])   //获取或设置text
$api.one(dom, 'click', function(){})    //只运行一次的事件
$api.setStorage(key, value)     //设置本地储存
$api.getStorage(key)    //获取本地储存
```
### api
```javascript
api.toast({msg:'msg',location:'middle'});   //弹出提示消息
api.execScript({    //运行其他win或frame的function
    name: 'root',   //name为打开win/frame设置的name
    script: 'function()'
});
api.openWin({   //打开win
    name: 'index',
    url: './index.html',
    reload: true,    //打开时刷新
    pageParam: {}   //可在新页面用api.pageParam获取传递的参数
})
api.closeWin({   //关闭win
    name: 'index'
});
api.openFrame({ //打开frame
    name: 'aa',
    url: './aa.html',
    bgColor : 'rgba(0,0,0,.2)',
    rect: { //frame的大小，位置
        x: 0,
        y: 0,
        w: api.winWidth,    //屏幕宽
        h: api.winHeight    //屏幕高
    },
    pageParam: {}
});
api.closeFrame({    //关闭对应frame
    name: 'aa'
})
//打开frameGroup
var frames = [{
            name: 'frame0',
            url: './frame0.html',
            bgColor : 'rgba(0,0,0,.2)',
            bounces:true
        },{
            name: 'frame1',
            url: './frame1.html',
            bgColor : 'rgba(0,0,0,.2)',
            bounces:true
        },
        //...
    ]
api.openFrameGroup({
    name: 'group',
    scrollEnabled: false,
    preload: 0, //预加载frame数
    rect: {
        x: 0,
        y: $api.dom('header').offsetHeight,
        w: api.winWidth,
        h: $api.dom('#main').offsetHeight   //除去header和footer的主体高度
    },
    index: 0,   //初始化后显示的ind
    frames: frames
}, function (ret, err) {
    //初始化后运行
});
//切换frameGroup中显示的frame
api.setFrameGroupIndex({
    name: 'group',
    index: 2,
    reload: true
});
//重新设置frameGroup的属性
api.setFrameGroupAttr({
    name: 'group',
    rect:{
        x: 0,
        y: 0,
        w: api.winWidth,
        h: api.winHeight
    }
})
//获取缓存大小
api.getCacheSize(function(ret) {
    var size = parseInt(ret.size);
    if(size >= 0){
        var result = (size/1024/1024).toFixed(1) + 'M'; //转为MB，保留一位小数
    }
})
//清除缓存
api.clearCache(function() {
    api.toast({msg: '清除完成'});
});
//调用电话
api.call({
    type: 'tel_prompt',
    number: '18788888888'
});
```

### 原生js
```javascript
window.location.reload();   //刷新当前页，传来的api.pageParam不会改变或消失
```


## 下方出现选择框
适用于选项较少的情况
```javascript
var buttons = ['男', '女'];
api.actionSheet({
    cancelTitle: '取消',
    buttons: buttons
}, function(ret, err) {
    if (ret) {
       var index = ret.buttonIndex; //index为buttons中选中项的index，从1开始
       var val = buttons[index - 1];
    } else {
        //alert(JSON.stringify(err));
    }
});
```


## 时间/日期选择框
```javascript
api.openPicker({
    type: 'date',
    title: '选择日期'
}, function(ret, err) {
    if (ret) {
       var data = ret.year + '-' + ret.month + '-' + ret.day;
    } else {
        //alert(JSON.stringify(err));
    }
});
```


## 下拉刷新
```javascript
api.setRefreshHeaderInfo({
    loadingImg: 'widget://image/refresh.png',
    bgColor: '#ccc',
    textColor: '#fff',
    textDown: '下拉刷新...',
    textUp: '松开刷新...'
}, function(ret, err) {
    // do something
    api.refreshHeaderLoadDone()
});
```


## 上拉加载
平台原生没这功能，这里是监听滚动条到底
```javascript
api.addEventListener({
    name: 'scrolltobottom',
    extra: {threshold:0}
}, function(ret, err){
    // do something
});
```


## 图片上传
```javascript
//图片上传
api.actionSheet({
    title : '上传照片',
    cancelTitle : '取消',
    buttons : ['拍照', '手机相册']
}, function(ret, err) {
    if (ret) {
        if (ret.buttonIndex == 1) { //拍照
            api.getPicture({
                sourceType : 'camera',
                encodingType : 'jpg',
                mediaValue : 'pic',
                destinationType : 'base64', //以base64格式给后端
                allowEdit : false,
                //quality : 100,
                saveToPhotoAlbum : true
            }, function(ret, err) {
                if (ret) {
                    saveImg(ret.base64Data);
                } else {
                    api.toast({msg : '图像获取失败',location : 'middle'});
                }
            });
        } else if (ret.buttonIndex == 2) {  //手机相册选图片
            api.getPicture({
                sourceType: 'library',
                encodingType: 'jpg',
                mediaValue: 'pic',
                destinationType: 'base64',
                allowEdit: false,
                //quality: 100,
                preview: true,
                saveToPhotoAlbum: false
            }, function(ret, err) {
                if (ret) {
                    saveImg(ret.base64Data);
                } else {
                    api.toast({msg : '图像获取失败',location : 'middle'});
                }
            });
        }
    }
});

//保存图片
function saveImg(base64Data) {
    api.showProgress({
        title: '上传中...',
        text: '先喝杯茶...',
    });
    //上传图像到服务器
    //项目中对ajax进行了封装，封装为$http，按实际情况修改
    $http({
        url: 'url',
        data: {
            picture: base64Data //作为参数传递给后台
        },
        successFn: function (res) { //封装后ajax成功的回调
            api.hideProgress();
            //图片上传后若需要回显，可以不用后台返回的url，直接显示base64数据
            //直接写在src上即可显示，<img src="base64Data">
        },
        failFn: function(){ //封装后ajax失败的回调
            api.hideProgress();
        }
    })
}
```


## 数字输入框

有时候需要接管输入框，在h5中的思路是给input设置disabled或readonly(否则会弹出原生输入框)，然后弹出自定义的输入框。

*需要注意的是这种方法的用户体验和原生比还是差了很多，相当于自己再设置了一个input，输入的内容再同步到disabled的input中。且原生的点击输入框界面上移(使输入框不被键盘挡住)、焦点在输入的input(此时该input已经disabled，无法获取焦点)等都暂时没找到实现。*

本项目中使用了mobiscroll，里面提供了数字输入框等非常多的UI组件，但该插件是收费的。通过某些方法拿到了版本较低（2.17.0）的[mobiscroll文件(js、css)][1]，该版本是基于jQ的，使用前需引入jQ。

[mobiscroll官方文档地址][2]，少部分用法和这个旧版本有些区别。

用法：
```javascript
//数字软键盘依赖
forInput(this, 'currentOrderNum', 'integer');

//数字输入框，允许两位小数，最多可输入6位
//key： 绑定的input的id
//type: integer(整数，最多9位字符),decimal1(一位小数，包括小数点最多5位字符)
function forInput(_this, key, type){
    switch(type){
        case 'integer':
            var template = 'ddddddddd';
            var maxLength = 9;
            var maxScale = 0;
            var leftButton = false;
            break;
        case 'decimal1':
            var template = 'ddddd';
            var maxLength = 5;
            var maxScale = 1;
            var leftButton = true;
            break;
    }
    var decimal = '.'
    //延迟加载是为了保证dom加载完再运行，避免找不到dom而失效
    setTimeout(function(){
        var dom = $('#' + key);
        dom.mobiscroll().numpad({
            theme: 'ios',   //皮肤
            lang: 'zh',
            headerText: '',  //键盘title显示的文字
            template: template, //几个d表示最多可以输入多少位
            maxLength: maxLength,
            maxScale: maxScale,	//最大小数位
            allowLeadingZero: true,
            parseValue: function (value) {
                if (value) {
                    return value;
                }
                return [];
            },
            formatValue: function (numbers) {	//接收值
                var ret = '',
                    l = numbers.length,
                    decimals,
                    i;

                // 第一个输入的是'.'时前面补0
                if (numbers[0] == '.') {
                    numbers.unshift(0);
                    l++;
                }

                return numbers.join('');
            },
            leftButton: leftButton ? {
                text: '.',
                value: '.'
            } : '',
            onBeforeShow: function(values, inst){
                if(!dom.val()){
                    dom.mobiscroll('setVal', '');
                }
            },
            onBeforeClose: function(values, inst){
                //点击确定时
                if(inst == 'set'){
                    _this[key] = values;
                }
            },
            validate: function (values, event, inst) {
                var s = inst.settings,
                    disabledButtons = [],
                    invalid = false;

                //有'.'的情况下禁用'.'
                if (values.length >= s.maxLength || values.indexOf(decimal) !== -1) {
                    disabledButtons.push(decimal);
                }
                //只有'0'时禁用1-9，只能输入'0'或'.'
                if (type !== 'integer' && values.length == 1 && values[0] === 0) {
                    for (var i = 1; i <= 9; i++) {
                        disabledButtons.push(i);
                    }
                }
                //为空或为'xxx.'的情况下，不能点'确定'
                if (!values.length || values[values.length - 1] == decimal) {
                    invalid = true;
                }

                // 达到最大小数位时无法输入
                if (values.length > (s.maxScale + 1) && values[values.length - s.maxScale - 1] == decimal) {
                    for (var i = 0; i <= 9; i++) {
                        disabledButtons.push(i);
                    }
                }

                // Display the formatted value
                if (inst.isVisible()) {
                    $('.mbsc-np-dsp', inst._markup).html(inst.settings.formatValue(values) || '&nbsp;');
                }

                return {
                    invalid: invalid,
                    disabled: disabledButtons
                };

            }
        });
    },900)
}
```


  [1]: https://github.com/LinXiudong/markdownPicture/tree/master/%E5%89%8D%E7%AB%AF%E5%AD%A6%E4%B9%A0/apicloud%E5%BC%80%E5%8F%91%E6%B7%B7%E5%90%88app
  [2]: https://docs.mobiscroll.com/jquery/numpad#usage
