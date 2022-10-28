---
title: 如何上传本地的图片到一篇博客文章中？
categories: 
- blog学习

---

本地的md文件中插入了一些图片，上传到博客中，图片却没加载出来，控制台404错误。

```
Failed to load resource: the server responded with a status of 404 (Not Found)
```

如何在在博客中插入图片？

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/imgs/blog学习/图片上传/bighead.jpg" width="40%" height="40%">
    <br>
</center>

<!--more-->

### MarkDown 中图片插入

**首先，要说到markdown文件中图片的插入格式。**

一般，我在做笔记的时候，插入图片的方法是直接将图片通过拖拽的方式插入到md文件中，这样在md中就能直接看到插入的图片效果。

但和doc文件不一样，md文件并不是通过将整个图片插入到文章中来实现图片的展示的，而是通过在文章中插入图片的存放路径来找到资源、间接展示，可以想到，这样的方式和html文件中插入图片的方式是一致的（ 事实上，md也支持html语法）。这就解释了两个问题：

> 1、为什么一篇文章中明明插入了很多张图片，实际大小确实只有几十kb？

因为文件中只存放了图片路径，也就是只有文字，没有图片数据。



> 2、为什么过一段时间，我的文章中的图片莫名其妙就裂开了？

可能因为你写文章时用的图片是从某个地方随意摘取的，清理数据或者整理文件夹时，图片的存放位置改变或者被删除，但是文章中存放的路径没变，相当于刻舟求剑了。

1.  

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/imgs/blog学习/图片上传/图片裂开.png">
    <br>
	<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">这是md文件中图片丢失的样子</div>
</center>



2.

![错误路径](/imgs/blog学习/图片上传/错误路径.png)

<center>  
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">这是web页面中图片丢失的样子</div>
</center>



md中的图片插入的语法为

```
![图像的替换文本](文件路径/图片名.png)
```

一般来说，通过拖拽方式插入的图片，文件路径一般是图片的**绝对路径**，如

```
![图像的替换文本](D:\文件夹\图片名.png) 
```

还有一种，是通过**相对路径** 

```
![图像的替换文本](..\图片文件夹\图片名.png) 
```

具体路径解析：

比如这篇文章的存放地址是 D: \项目文件夹\文章文件夹\文章.md

而文章中用到的图片的位置是D: \项目文件夹\图片文件夹\图片.jpg

那么使用相对路径时，就用" ..\ "来代表上一级文件夹，藏得越深，"..\"越多。



### typora图片相对路径设置

首先一个问题，html中文件路径是左斜杠/，md绝对路径用的是右斜杠\，这就导致如果想要在博客中展示图片，可能会使本地文件中看不到图片效果。

本人使用的解决方法为：在typora（编辑md文件使用的软件）中，设置图片根目录。设置的地方是格式→图像→设置图片根目录。然后把根目录设置为图片文件夹所在的地方，在文件中，图片的插入方式就是\[图像的替换文本](/文件路径/图片名.png)，上传后，博客中图片也能正常显示了。

**注意，这种方法对每一个md文件都是独立的，而且与最后介绍的另一种方法冲突。**

eg. 下图路径：/imgs/blog学习/图片上传/设置路径.png

<img src="/imgs/blog学习/图片上传/设置路径.png" alt="设置路径" style="zoom:50%;" />



### 博客中图片插入

再说回来，要想让同一路径下，md和网页的图片都正常显示，就还需要进行一些设置。

首先，对博客文件夹下的\_config.yml做修改，将其中一个属性改为true

```
post_asset_folder: true  //原为false
```

然后安装一个插件

```
npm install https://github.com/CodeFalling/hexo-asset-image --save
```

并在hexoblog\node_modules\hexo-asset-image 中将index.js文件内容替换为如下代码

```
'use strict';
var cheerio = require('cheerio');

// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
function getPosition(str, m, i) {
  return str.split(m, i).join(m).length;
}

var version = String(hexo.version).split('.');
hexo.extend.filter.register('after_post_render', function(data){
  var config = hexo.config;
  if(config.post_asset_folder){
    	var link = data.permalink;
	if(version.length > 0 && Number(version[0]) == 3)
	   var beginPos = getPosition(link, '/', 1) + 1;
	else
	   var beginPos = getPosition(link, '/', 4) + 1;     //markdown的本地路径是xxx/x.jpg    hexo g编译后加上这句就是public/2020/05/26/xxx/xx.jpg 前4个文件+后一个图片~
	// In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
	var endPos = link.lastIndexOf('/') + 1;
    link = link.substring(beginPos, endPos);

    var toprocess = ['excerpt', 'more', 'content'];
    for(var i = 0; i < toprocess.length; i++){
      var key = toprocess[i];
 
      var $ = cheerio.load(data[key], {
        ignoreWhitespace: false,
        xmlMode: false,
        lowerCaseTags: false,
        decodeEntities: false
      });

      $('img').each(function(){
		if ($(this).attr('src')){
			// For windows style path, we replace '\' to '/'.
			var src = $(this).attr('src').replace('\\', '/');
			if(!/http[s]*.*|\/\/.*/.test(src) &&
			   !/^\s*\//.test(src)) {
			  // For "about" page, the first part of "src" can't be removed.
			  // In addition, to support multi-level local directory.
			  var linkArray = link.split('/').filter(function(elem){
				return elem != '';
			  });
			  var srcArray = src.split('/').filter(function(elem){
				return elem != '' && elem != '.';
			  });
			  if(srcArray.length > 1)
				srcArray.shift();
			  src = srcArray.join('/');
			  $(this).attr('src', config.root + link + src);
			  console.info&&console.info("update link as:-->"+config.root + link + src);
			}
		}else{
			console.info&&console.info("no src attr, skipped...");
			console.info&&console.info($(this));
		}
      });
      data[key] = $.html();
    }
  }
});
```

然后博客中图片也可以正常显示了。

### 其他方法

但其实，上面的做法还是有点繁琐，比如图片文件与文章不在同一目录下，当文章数量变多时，插入图像就变成了比较麻烦的操作。

目标是，在文章同级文件夹下建立一个同名文件夹，使得md本地预览和网页都能正常显示。

折腾了一个下午和晚上，没找到理想方法，但下面这种算是接近。

要点：

1. 设置post_asset_folder: true  //原为false

2. hexoblog\node_modules\hexo-asset-image 中将index.js文件内容替换代码

3. 图片插入格式为\![image]\(image.jpg) ，而非\![image]\(文章名/image.jpg) 。这样在博客中能正常展示，在md中虽然不能展示，但是加个文件名也能看到。此时不能设置文件根路径，否则本地也看不见。

4. 对typora进行设置：文件→偏好设置→图像→复制到指定路径→./{$filename}，功能和上面一样，但是不在hexo中编辑时同样有用。

5. 也可以是

	```
	{% asset_img  panda.jpg  image text %}
	```

6. 附加一个图片注解方式

	```
	<center>
	    <img style="border-radius: 0.3125em;
	    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
	    src="地址">
	    <br>
		<div style="color:orange; border-bottom: 1px solid #d9d9d9;
	    display: inline-block;
	    color: #999;
	    padding: 2px;">注解</div>
	</center>
	```

### 问题

主要是图片大小控制的问题

有两种方式

1. 在asset_img外加div标签对，这个方法对md式\![]\()无效。

```
<div style="width:40%;margin:auto;">{% asset_img bighead.jpg 图片信息描述 %}</div>
```

2. 使用html的\<img>标签，但是src中似乎只能填以source为根目录的额外建的imgs文件夹中的图片路径，直接使用图片名或者用文章同名文件夹中的图片都无效。

```
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/imgs/blog学习/图片上传/bighead.jpg" width="40%" height="40%">
    <br>
</center>

```

1. bighead.jpg
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="bighead.jpg" width="40%" height="40%">
    <br>
</center>

2.  \_posts\blog\博客以及md文件的图片上传\bighead.jpg 
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src=" \_posts\blog\博客以及md文件的图片上传\bighead.jpg " width="40%" height="40%">
    <br>
</center>

3. /_posts/blog/博客以及md文件的图片上传/bighead.jpg
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/_posts/blog/博客以及md文件的图片上传/bighead.jpg" width="40%" height="40%">
    <br>
</center>



### 总结

1、使用从source开始的相对路径+自己在source下建立一个img文件夹，在typora中设置图片根目录，加上上面几项设置。通过

```
![代理图片名](/imgs/图片.png)
```

能在文件和文章中看到同时看到图片。

2、设置文章同名文件夹，方便管理，上传网站时从图片路径中删去文件名，推荐使用。

![panda.jpg](panda.jpg)


关于控制图片大小，以及插入的问题，还需要再看看。

### 参考文章

[解决了hexo图像，但是md不显示](https://blog.csdn.net/as3522/article/details/102972473)

[同上，但分析较全](https://blog.csdn.net/m0_43401436/article/details/107191688?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)
