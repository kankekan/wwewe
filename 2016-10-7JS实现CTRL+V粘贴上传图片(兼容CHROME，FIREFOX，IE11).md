---
title: 2016-10-7JS实现CTRL+V粘贴上传图片(兼容CHROME，FIREFOX，IE11)
tags: 技术,小书匠
grammar_cjkRuby: true
---

## 背景

我们或多或少都使用过各式各样的富文本编辑器，其中有一个很方便功能，复制一张图片然后粘贴进文本框，这张图片就被上传了，那么这个方便的功能是如何实现的呢？

## 原理分析

提取操作：复制=>粘贴=>上传  
在这个操作过程中，我们需要做的就是：监听粘贴事件=>获取剪贴板里的内容=>发请求上传  
为方便理解下文，需要先明白几点：

1. 我们只能上传网页图(在网页上右键图片，然后复制)和截图(截图工具截的图片，eg：qq截图)，不能粘贴上传系统里的图片(从桌面上、硬盘里复制)，他们是存在完全不同的地方的。
1. 截图工具截的图与在网页点击右键复制的图是有些不同的，因此处理方式也不一样。
1. 知悉paste event这个事件：当进行粘贴（右键paste/ctrl+v）操作时，该动作将触发名为’paste’的剪贴板事件，这个事件的触发是在剪贴板里的数据插入到目标元素之前。如果目标元素（光标所在位置）是可编辑的元素（eg：设置了contenteditable属性的div。textarea并不行。），粘贴动作将把剪贴板里的数据，以最合适的格式，插入到目标元素里；如果目标元素不可编辑，则不会插入数据，但依然触发paste event。数据在粘贴的过程中是只读的。
可惜的是，经过试验，发现chrome(当前最新版)、firefox(当前最新版)、ie11对paste事件的实现并不是完全按照w3c来的，各自也有区别(w3c的paste标准也因此只是草案阶段)。

test代码及截图如下:

#### chrome：

``` html
<textarea ></textarea>  
<div contenteditable style="width: 100px;height: 100px; border:1px solid">  
</div>  
<script>  
document.addEventListener('paste', function (event) {  
    console.log(event)  
})
</script> 
```

![enter description here][1]

![enter description here][2]

![enter description here][3]

![enter description here][4]

![enter description here][5]

![enter description here][6]

* event有clipboardData属性，且clipboardData有item属性，clipboardData.item中的元素（对象）有type和kind属性；
* 无论在哪进行粘贴，均可触发paste事件；
* 在div（未特殊声明时，本文div均指设置了contenteditable属性的div） 里粘贴截图，不显示图片。
* 在div里粘贴网页图片，直接显示图片，img.src为图片地址。

firefox：


![enter description here][7]

![enter description here][8]

![enter description here][9]

![enter description here][10]

* event有clipboardData属性，clipboardData没有item属性；
* 只有在textarea里或者div里粘贴才触发paste事件；
* 在div里粘贴截图，直接显示图片，img.src为base64编码字符串；
* 在div里粘贴网页图片，表现同chrome。

#### ie11：（不截图了，可自行试验，其他浏览器同理<(_￣▽￣_)/）

* event没有clipboardData属性；
* 只在div里粘贴才触发paste事件；
* 在div里粘贴截图，直接显示图片，img.src为base64编码字符串；
* 在div里粘贴网页图片，表现同chrome。

## 获取数据

监听了paste事件，也知道了表现形式，接下来就是如何获取数据了：  
chrome有特定的方法，利用clipboardData.items、getAsFile()、new FileReader()等api可以在paste回调函数里获取到剪贴板里图片的base64编码字符串（无论是截图粘贴的还是网页图片复制粘贴的），ie11，firefox没有这样的api，不过依然有办法可以获取，因为数据已经表现在img的src里了，对于截图粘贴的，直接取img的src属性值（base64），对于网页粘贴的，则把地址传给后台，然后根据地址down下来，存在自己的服务器，最后把新地址返回来交给前端展示就ok了。为了保持一致性便于管理，统一将所有情况（截图、网页）中的img的src属性替换为自己存储的地址。因此可以得到以下核心代码（注释很全哦~~）：

html展示：

``` html
<head>
<meta charset="UTF-8">
<title>Document</title>
<style>
body {
    display: -webkit-flex; 
    display: flex;       
    -webkit-justify-content: center;
    justify-content: center;
}
#tar_box {
    width: 500px;
    height: 500px;
    border: 1px solid red;
}
</style>
```

前端js处理逻辑:

``` javascript
document.addEventListener('paste', function (event) {
    console.log(event)
    var isChrome = false;
    if ( event.clipboardData || event.originalEvent ) {
        //not for ie11   某些chrome版本使用的是event.originalEvent
        var clipboardData = (event.clipboardData || event.originalEvent.clipboardData);
        if ( clipboardData.items ) {
            // for chrome
            var    items = clipboardData.items,
                len = items.length,
                blob = null;
            isChrome = true;
            //items.length比较有意思，初步判断是根据mime类型来的，即有几种mime类型，长度就是几（待验证）
            //如果粘贴纯文本，那么len=1，如果粘贴网页图片，len=2, items[0].type = 'text/plain', items[1].type = 'image/*'
            //如果使用截图工具粘贴图片，len=1, items[0].type = 'image/png'
            //如果粘贴纯文本+HTML，len=2, items[0].type = 'text/plain', items[1].type = 'text/html'
            // console.log('len:' + len);
            // console.log(items[0]);
            // console.log(items[1]);
            // console.log( 'items[0] kind:', items[0].kind );
            // console.log( 'items[0] MIME type:', items[0].type );
            // console.log( 'items[1] kind:', items[1].kind );
            // console.log( 'items[1] MIME type:', items[1].type );

            //阻止默认行为即不让剪贴板内容在div中显示出来
            event.preventDefault();

            //在items里找粘贴的image,据上面分析,需要循环   
            for (var i = 0; i < len; i++) {
                if (items[i].type.indexOf("image") !== -1) {
                    // console.log(items[i]);
                    // console.log( typeof (items[i]));

                    //getAsFile()  此方法只是living standard  firefox ie11 并不支持               
                    blob = items[i].getAsFile();
                }
            }
            if ( blob !== null ) {
                var reader = new FileReader();
                reader.onload = function (event) {
                    // event.target.result 即为图片的Base64编码字符串
                    var base64_str = event.target.result
                    //可以在这里写上传逻辑 直接将base64编码的字符串上传（可以尝试传入blob对象，看看后台程序能否解析）
                    uploadImgFromPaste(base64_str, 'paste', isChrome);
                }
                reader.readAsDataURL(blob); 
            }
        } else {
            //for firefox
            setTimeout(function () {
                //设置setTimeout的原因是为了保证图片先插入到div里，然后去获取值
                var imgList = document.querySelectorAll('#tar_box img'),
                    len = imgList.length,
                    src_str = '',
                    i;
                for ( i = 0; i < len; i ++ ) {
                    if ( imgList[i].className !== 'my_img' ) {
                        //如果是截图那么src_str就是base64 如果是复制的其他网页图片那么src_str就是此图片在别人服务器的地址
                        src_str = imgList[i].src;
                    }
                }
                uploadImgFromPaste(src_str, 'paste', isChrome);
            }, 1);
        }
    } else {
        //for ie11
        setTimeout(function () {
            var imgList = document.querySelectorAll('#tar_box img'),
                len = imgList.length,
                src_str = '',
                i;
            for ( i = 0; i < len; i ++ ) {
                if ( imgList[i].className !== 'my_img' ) {
                    src_str = imgList[i].src;
                }
            }
            uploadImgFromPaste(src_str, 'paste', isChrome);
        }, 1);
    }
})

function uploadImgFromPaste (file, type, isChrome) {
    var formData = new FormData();
    formData.append('image', file);
    formData.append('submission-type', type);

    var xhr = new XMLHttpRequest();
    xhr.open('POST', '/upload_image_by_paste');
    xhr.onload = function () {
        if ( xhr.readyState === 4 ) {
            if ( xhr.status === 200 ) {
                var data = JSON.parse( xhr.responseText ),
                    tarBox = document.getElementById('tar_box');
                if ( isChrome ) {
                    var img = document.createElement('img');
                    img.className = 'my_img';
                    img.src = data.store_path;
                    tarBox.appendChild(img);
                } else {
                    var imgList = document.querySelectorAll('#tar_box img'),
                        len = imgList.length,
                        i;
                    for ( i = 0; i < len; i ++) {
                        if ( imgList[i].className !== 'my_img' ) {
                            imgList[i].className = 'my_img';
                            imgList[i].src = data.store_path;
                        }
                    }
                }

            } else {
                console.log( xhr.statusText );
            }
        };
    };
    xhr.onerror = function (e) {
        console.log( xhr.statusText );
    }
    xhr.send(formData);
}
```

用express.js搭的简易后台的接收逻辑：

``` javascript
router.post('/', upload.array(), function (req, res, next) {
//1.获取客户端传来的src_str字符串=>判断是base64还是普通地址=>获取图片类型后缀(jpg/png etc)
//=>如果是base64替换掉"前缀"("data:image\/png;base64," etc)
//2.base64 转为 buffer对象  普通地址则先down下来
//3.写入硬盘(后续可以将地址存入数据库)
//4.返回picture地址
var src_str = req.body.image,
    timestamp = new Date().getTime();
if ( src_str.match(/^data:image\/png;base64,|^data:image\/jpg;base64,|^data:image\/jpg;base64,|^data:image\/bmp;base64,/) ) {
    //处理截图 src_str为base64字符串
    var pic_suffix = src_str.split(';',1)[0].split('/',2)[1],
        base64 = src_str.replace(/^data:image\/png;base64,|^data:image\/jpg;base64,|^data:image\/jpg;base64,|^data:image\/bmp;base64,/, ''),
        buf = new Buffer(base64, 'base64'),
        store_path = 'public/images/test_' + timestamp + '.' + pic_suffix;

    fs.writeFile(store_path, buf, function (err) {
        if (err) {
            throw err;
        } else {
            res.json({'store_path': store_path});
        }
    });
} else {// 处理非chrome的网页图片 src_str为图片地址
    var temp_array = src_str.split('.'),
        pic_suffix = temp_array[temp_array.length - 1],
        store_path = 'public/images/test_' + timestamp + '.' + pic_suffix,
        wstream = fs.createWriteStream(store_path);

    request(src_str).pipe(wstream);
    wstream.on('finish', function (err) {
        if( err ) {
            throw err;
        } else {
            res.json({"store_path": store_path});
        }
    });
}
});
```

完整demo，请访问渣渣github下载..（跑起来需要node环境：安装node=>npm intall=>node app.js)
第一篇文章终于拼凑完了….(╯﹏╰)b 欢迎大大们拍砖o(╯□╰)o

巨人们的肩膀：

> https://dzone.com/articles/paste-wasteland-or-why-onpaste
http://strd6.com/2011/09/html5-javascript-pasting-image-data-in-chrome/http://stackoverflow.com/questions/6333814/how-does-the-paste-image-from-clipboard-functionality-work-in-gmail-and-google-c
http://stackoverflow.com/questions/18055422/how-to-receive-php-image-data-over-copy-n-paste-javascript-with-xmlhttpreques


http://www.cnphp6.com/archives/135763


  [1]: ./images/1475848053988.jpg "1475848053988.jpg"
  [2]: ./images/1475848080260.jpg "1475848080260.jpg"
  [3]: ./images/1475848088823.jpg "1475848088823.jpg"
  [4]: ./images/1475848096759.jpg "1475848096759.jpg"
  [5]: ./images/1475848105743.jpg "1475848105743.jpg"
  [6]: ./images/1475848113795.jpg "1475848113795.jpg"
  [7]: ./images/1475848144760.jpg "1475848144760.jpg"
  [8]: ./images/1475848152531.jpg "1475848152531.jpg"
  [9]: ./images/1475848160774.jpg "1475848160774.jpg"
  [10]: ./images/1475848166890.jpg "1475848166890.jpg"