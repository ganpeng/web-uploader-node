web-uploader-node
=================

js (html5 + html4) 文件上传管理器，支持上传进度显示，支持 IE6+、Firefox、Chrome等 - node示例

###特点：
<ul>
	<li>轻量级，不依赖任何JS库，核心代码（Q.Uploader.js）仅约700行，min版本加起来不到12KB</li>
	<li>纯JS代码，无需Flash，无需更改后台代码即可实现带进度条（IE10+、其它标准浏览器）的上传，其它（eg：IE6+）自动降级为传统方式上传</li>
	<li>上传核心与UI界面分离，可以很方便的定制上传界面包括上传按钮</li>
	<li>上传文件的同时可以指定上传参数，支持上传类型过滤</li>
	<li>完善的事件回调，可针对上传的每个过程进行单独处理</li>
	<li>方便的UI接口，上传界面可以随心所欲的定制</li>
</ul>

###演示环境（其它语言可自行实现服务端接收和网站部署）
演示前请在根目录下创建upload文件夹，以保存上传文件<br>
执行程序：app.js   上传处理：/upload
```
1. 执行演示程序前请确保已安装node环境(http://www.nodejs.org/download/)
2. 命令行执行 node app.js ，windows 环境可双击运行 start.bat 来快速启动程序
3. 打开浏览器访问 http://127.0.0.1:3000/

```

###简单调用示例
例：使用默认的UI
```
1. 导入样式文件(若自己实现UI接口，则无需导入默认的样式文件)
<link href="css/uploader.css" rel="stylesheet" type="text/css" />

2. 导入js文件（可自行合并，若自己实现UI接口，则无需导入 Q.Uploader.UI.js 文件）
<script type="text/javascript" src="js/Q.js"></script>
<script type="text/javascript" src="js/Q.Uploader.js"></script>
<script type="text/javascript" src="js/Q.Uploader.UI.js"></script>

3. 调用
new Q.Uploader({
	url:"api/upload.ashx",

	target: element,    //上传按钮
	view: element       //上传任务视图
});
```

###完整调用示例
```
new Q.Uploader({
	//--------------- 必填 ---------------
	url: "",            //上传路径
	target: element,    //上传按钮
	view: element,      //上传任务视图(若自己实现UI接口，则无需指定此参数)

	//--------------- 可选 ---------------
	html5: true,       //是否启用html5上传,若浏览器不支持,则自动禁用
	multiple: true,    //选择文件时是否允许多选,若浏览器不支持,则自动禁用(仅html5模式有效)

    clickTrigger:true, //是否启用click触发文件选择 eg: input.click() => ie9及以下不支持

	auto: true,        //添加任务后是否立即上传

	data: {},          //上传文件的同时可以指定其它参数,该参数将以POST的方式提交到服务器

	workerThread: 1,   //同时允许上传的任务数(仅html5模式有效)

	upName: "upfile",  //上传参数名称,若后台需要根据name来获取上传数据,可配置此项

	allows: "",        //允许上传的文件类型(扩展名),多个之间用逗号隔开
	disallows: "",     //禁止上传的文件类型(扩展名)

	container:element, //一般无需指定
	getPos:function,   //一般无需指定

	//上传回调事件(function)
	on: {
		init,          //上传管理器初始化完毕后触发
    
		select,        //点击上传按钮准备选择上传文件之前触发,返回false可禁止选择文件
		add,           //添加任务之前触发,返回false将跳过该任务
		upload,        //上传任务之前触发,返回false将跳过该任务
		send,          //发送数据之前触发,返回false将跳过该任务
    
		cancel,        //取消上传任务后触发
		remove,        //移除上传任务后触发
    
		progress,      //上传进度发生变化后触发(仅html5模式有效)
		complete       //上传完成后触发
	},

	//UI接口(function),若指定了以下方法,将忽略默认实现
	UI:{
		init,       //执行初始化操作
		draw,       //添加任务后绘制任务界面
		update,     //更新任务界面  
		over        //任务上传完成
	}
});
```

说明：回调事件(add、upload、send)支持异步调用，只需在后面加上Async即可，比如在上传之前需要访问服务器验证数据，通过的就上传，否则跳过
```
on: {
	uploadAsync: function (callback) {
        $.postJSON(url, function (json) {
            //若 json.ok 返回false，该任务不会上传
            callback(json.ok);
        });
    }
}
```

###自定义UI实现
可以在初始化时指定UI处理函数，亦可以通过扩展的方式实现
```
Uploader.extend({
    //初始化操作，一般无需处理
    init: function () { },

    //绘制任务界面
    draw: function (task) {
        //每个task就是一个上传任务，task一般有以下属性
        /*task = {
            id,         //任务编号

            name,       //上传文件名（包括扩展名）
            ext,        //上传文件扩展名
            size,       //上传文件大小（单位：Byte，若获取不到大小，则值为-1）

            input,      //上传控件
            file,       //上传数据（仅 html5）

            state,      //上传状态

			disabled,   //若为true，表示禁止上传的文件

			//上传后会有如下属性（由于浏览器支持问题，以下部分属性可能不存在）
			xhr,        //XMLHttpRequest对象（仅 html5）

			total,      //总上传大小（单位：Byte）
			loaded,     //已上传大小（单位：Byte）
			speed,      //上传速度（单位：Byte/s）

			avg_speed,  //平均上传速度（仅上传完毕）

			timeStart,  //开始上传的时间
			timeEnd,    //结束上传的时间（仅上传完毕）

			deleted     //若为true，表示已删除的文件
        };*/
    },

    //更新上传进度
    update: function (task) {

    },

    //上传完毕，一般无需处理
    over: function () { }
});
```