#### 1.且看需求
最近，景科同学碰到个需求，就是做一个图片【上传预览】功能。之前的做法都是先上传，后端返回图片地址给客户端。不过这次不一样了，或许叫【预览上传】可能比较贴切，上传之前需要用户先预览，然后还会再做一系列的操作，操作之后再和其他信息一起上传提交。功能不复杂，但是以前没用过，好记性不如小笔记，所以有了本文。
#### 2.概述
实现之前先来系统的了解一下本次的主角FileReader对象。
> 1. FileReader的使用非常简单，有如下4种方法可以供我们调用。

方法名 | 参数 | 描述
---|---|---
abort | none | 中断读取
readAsBinaryString | file | 将文件读取为二进制码
readAsDataURL | file | 	将文件读取为DataURL
readAsText | file | 将文件读取为文本
readAsArrayBuffer | file | 将文件读取为ArrayBuffer

```javascript
1. readAsBinaryString：该方法将文件读取为二进制字符串，通常我们将它传送到后端，后端可以通过这段字符串存储文件。
2. readAsDataURL：这是实现本文需求所用的方法，该方法将文件读取为一段以data: 开头的字符串，这段字符串的实质就是 Data URL，Data URL是一种将小文件直接嵌入文档的方案。这里的小文件通常是指图像与 html 等格式的文件。
3. readAsText：该方法有两个参数，其中第二个参数是文本的编码方式，默认值为 UTF-8。这个方法非常容易理解，将文件以文本方式读取，读取的结果即是这个文本文件中的内容。
```
> 2. FileReader有如下6种事件处理程序

方法名 | 描述
---|---
onloadstart | 开始读取
onprogress | 读取中
onload | 读取完成（成功）
onloadend | 读取完成（成功、失败）
onabort | 读取中断
onerror | 读取出错
```javascript
// 文件读取结果将会赋值给this.result,当然也可以从接口底层返回的参数中得到，实现如下：
reader.onload = function (event) {
    // 注：读取无论成功还是失败，result都会被赋值，如果读取失败，则 result 的值为 null
    console.log(this.result);
    // 读取结果
    console.log(event);
    // ProgressEvent {isTrusted: true, lengthComputable: true, loaded: 346, total: 346, type: "load",…}
}
```
#### 3. 具体使用
> 由于比较简单，直接贴代码
```javascript
let oFile = document.querySelector('.file-upload');
oFile.addEventListener('change', onInputChange, false);
function onInputChange () {
	if (!window.FileReader) return;
	var rFilter = /^(?:image\/bmp|image\/cis\-cod|image\/gif|image\/ief|image\/jpeg|image\/jpeg|image\/jpeg|image\/pipeg|image\/png|image\/svg\+xml|image\/tiff|image\/x\-cmu\-raster|image\/x\-cmx|image\/x\-icon|image\/x\-portable\-anymap|image\/x\-portable\-bitmap|image\/x\-portable\-graymap|image\/x\-portable\-pixmap|image\/x\-rgb|image\/x\-xbitmap|image\/x\-xpixmap|image\/x\-xwindowdump)$/i;
	let file = oFile.files[0];
	if (!rFilter.test(file.type)) {
			console.log("请选择图片格式的文件");
			return;
	}
	let reader = new FileReader();
	reader.onloadstart = function () {
		console.log('step1: 开始读取...');
	}
	reader.onprogress = function () {
		console.log('step2: 正在读取...');
	}
	reader.onload = function (event) {
		console.log('step3: 读取完成...');
		// 在此处处理
		onReaderLoad(file, this.result);
	}
	reader.onloadend = function () {
		console.log('step4: 读取结束...');
	}
	reader.onabort = function () {
		console.log('读取中断...');
	}
	reader.onerror = function () {
		console.log('读取失败...');
	}
	reader.readAsDataURL(file);
	// reader.readAsBinaryString(file);
	// reader.readAsText(file, 'utf8');
}
function onReaderLoad (file, result) {
	// 用于显示
	let oImg = document.querySelector('.preview');
	oImg.setAttribute('src', result);
	// 整理用于ajax上传
	var fd = new FormData();
    fd.append("file", file);
    console.log(file);
     console.log(result);
}
```
> 至此，本文完！





