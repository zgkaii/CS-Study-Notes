## 今日内容：
	1. JQuery 高级
		1. 动画
		2. 遍历
		3. 事件绑定
		4. 案例
		5. 插件



## JQuery 高级
	1. 动画
		1. 三种方式显示和隐藏元素
			1. 默认显示和隐藏方式
				1. show([speed,[easing],[fn]])
					1. 参数：
						1. speed：动画的速度。三个预定义的值("slow","normal", "fast")或表示动画时长的毫秒数值(如：1000)
						2. easing：用来指定切换效果，默认是"swing"，可用参数"linear"
							* swing：动画执行时效果是 先慢，中间快，最后又慢
							* linear：动画执行时速度是匀速的
						3. fn：在动画完成时执行的函数，每个元素执行一次。
	
				2. hide([speed,[easing],[fn]])
				3. toggle([speed],[easing],[fn])
			
			2. 滑动显示和隐藏方式
				1. slideDown([speed],[easing],[fn])
				2. slideUp([speed,[easing],[fn]])
				3. slideToggle([speed],[easing],[fn])
	
			3. 淡入淡出显示和隐藏方式
				1. fadeIn([speed],[easing],[fn])
				2. fadeOut([speed],[easing],[fn])
				3. fadeToggle([speed,[easing],[fn]])

	2. 遍历
		1. js的遍历方式
			* for(初始化值;循环结束条件;步长)
		2. jq的遍历方式
			1. jq对象.each(callback)
				1. 语法：
					jquery对象.each(function(index,element){});
						* index:就是元素在集合中的索引
						* element：就是集合中的每一个元素对象
	
						* this：集合中的每一个元素对象
				2. 回调函数返回值：
					* true:如果当前function返回为false，则结束循环(break)。
					* false:如果当前function返回为true，则结束本次循环，继续下次循环(continue)
			2. $.each(object, [callback])
			3. for..of: jquery 3.0 版本之后提供的方式
				for(元素对象 of 容器对象)
		
	3. 事件绑定
		1. jquery标准的绑定方式
			* jq对象.事件方法(回调函数)；
			* 注：如果调用事件方法，不传递回调函数，则会触发浏览器默认行为。
				* 表单对象.submit();//让表单提交
		2. on绑定事件/off解除绑定
			* jq对象.on("事件名称",回调函数)
			* jq对象.off("事件名称")
				* 如果off方法不传递任何参数，则将组件上的所有事件全部解绑
		3. 事件切换：toggle
			* jq对象.toggle(fn1,fn2...)
				* 当单击jq对象对应的组件后，会执行fn1.第二次点击会执行fn2.....
				
			* 注意：1.9版本 .toggle() 方法删除,jQuery Migrate（迁移）插件可以恢复此功能。
				 <script src="../js/jquery-migrate-1.0.0.js" type="text/javascript" charset="utf-8"></script>

	4. 案例
		1. 广告显示和隐藏
			<!DOCTYPE html>
			<html>
			<head>
			    <meta charset="UTF-8">
			    <title>广告的自动显示与隐藏</title>
			    <style>
			        #content{width:100%;height:500px;background:#999}
			    </style>
			
			    <!--引入jquery-->
			    <script type="text/javascript" src="../js/jquery-3.3.1.min.js"></script>
			    <script>
			        /*
			            需求：
			                1. 当页面加载完，3秒后。自动显示广告
			                2. 广告显示5秒后，自动消失。
			
			            分析：
			                1. 使用定时器来完成。setTimeout (执行一次定时器)
			                2. 分析发现JQuery的显示和隐藏动画效果其实就是控制display
			                3. 使用  show/hide方法来完成广告的显示
			         */
			
			        //入口函数，在页面加载完成之后，定义定时器，调用这两个方法
			        $(function () {
			           //定义定时器，调用adShow方法 3秒后执行一次
			           setTimeout(adShow,3000);
			           //定义定时器，调用adHide方法，8秒后执行一次
			            setTimeout(adHide,8000);
			        });
			        //显示广告
			        function adShow() {
			            //获取广告div，调用显示方法
			            $("#ad").show("slow");
			        }
			        //隐藏广告
			        function adHide() {
			            //获取广告div，调用隐藏方法
			            $("#ad").hide("slow");
			        }


​			
			    </script>
			</head>
			<body>
			<!-- 整体的DIV -->
			<div>
			    <!-- 广告DIV -->
			    <div id="ad" style="display: none;">
			        <img style="width:100%" src="../img/adv.jpg" />
			    </div>
			
			    <!-- 下方正文部分 -->
			    <div id="content">
			        正文部分
			    </div>
			</div>
			</body>
			</html>


		2. 抽奖
			<!DOCTYPE html>
			<html>
			<head>
			    <meta charset="UTF-8">
			    <title>jquery案例之抽奖</title>
			    <script type="text/javascript" src="../js/jquery-3.3.1.min.js"></script>
			
			    <script language='javascript' type='text/javascript'>
			
			        /*
			            分析：
			                1. 给开始按钮绑定单击事件
			                    1.1 定义循环定时器
			                    1.2 切换小相框的src属性
			                        * 定义数组，存放图片资源路径
			                        * 生成随机数。数组索引


			                2. 给结束按钮绑定单击事件
			                    1.1 停止定时器
			                    1.2 给大相框设置src属性
			
			         */
			        var imgs = ["../img/man00.jpg",
			                    "../img/man01.jpg",
			                    "../img/man02.jpg",
			                    "../img/man03.jpg",
			                    "../img/man04.jpg",
			                    "../img/man05.jpg",
			                    "../img/man06.jpg",
			                    ];
			        var startId;//开始定时器的id
			        var index;//随机角标
			        $(function () {
			            //处理按钮是否可以使用的效果
			            $("#startID").prop("disabled",false);
			            $("#stopID").prop("disabled",true);


			           //1. 给开始按钮绑定单击事件
			            $("#startID").click(function () {
			                // 1.1 定义循环定时器 20毫秒执行一次
			                startId = setInterval(function () {
			                    //处理按钮是否可以使用的效果
			                    $("#startID").prop("disabled",true);
			                    $("#stopID").prop("disabled",false);


			                    //1.2生成随机角标 0-6
			                    index = Math.floor(Math.random() * 7);//0.000--0.999 --> * 7 --> 0.0-----6.9999
			                    //1.3设置小相框的src属性
			                    $("#img1ID").prop("src",imgs[index]);
			
			                },20);
			            });


			            //2. 给结束按钮绑定单击事件
			            $("#stopID").click(function () {
			                //处理按钮是否可以使用的效果
			                $("#startID").prop("disabled",false);
			                $("#stopID").prop("disabled",true);


			               // 1.1 停止定时器
			                clearInterval(startId);
			               // 1.2 给大相框设置src属性
			                $("#img2ID").prop("src",imgs[index]).hide();
			                //显示1秒之后
			                $("#img2ID").show(1000);
			            });
			        });


​			
​			
			    </script>
			
			</head>
			<body>
			
			<!-- 小像框 -->
			<div style="border-style:dotted;width:160px;height:100px">
			    <img id="img1ID" src="../img/man00.jpg" style="width:160px;height:100px"/>
			</div>
			
			<!-- 大像框 -->
			<div
			        style="border-style:double;width:800px;height:500px;position:absolute;left:500px;top:10px">
			    <img id="img2ID" src="../img/man00.jpg" width="800px" height="500px"/>
			</div>
			
			<!-- 开始按钮 -->
			<input
			        id="startID"
			        type="button"
			        value="点击开始"
			        style="width:150px;height:150px;font-size:22px">
			
			<!-- 停止按钮 -->
			<input
			        id="stopID"
			        type="button"
			        value="点击停止"
			        style="width:150px;height:150px;font-size:22px">


			</body>
			</html>
	
	5. 插件：增强JQuery的功能
		1. 实现方式：
			1. $.fn.extend(object) 
				* 增强通过Jquery获取的对象的功能  $("#id")
			2. $.extend(object)
				* 增强JQeury对象自身的功能  $/jQuery
