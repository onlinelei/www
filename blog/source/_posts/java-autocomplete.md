---
title: autocomplete --小例
date: 2016-07-09 17:59:44
tags: java
---


###最近项目使用到autocomplete的功能，使用中关于编码遇到了点问题，这里分享下   
>AutoComplete控件就是指用户在文本框输入前几个字母或是汉字的时候，该控件就能从存放数据的文本或是数据库里将所有以这些字母开头的数据提示给用户，供用户选择，提供方便。   
<!--more-->

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2007394-61142480e6033351.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
html代码

	姓名：<input type="text"  id="kid" name="kiName">


jquery 代码：

	$(function(){
			$("#kid").autocomplete(
					"${pageContext.request.contextPath}/kids/selectKidAutoComplete.do",
				{
				minChars : 1, //最少的查询字符数
				width: 200,     //提示的宽度，溢出隐藏
				scrollHeight: 300,   //提示的高度，溢出显示滚动条
				matchContains: true,    //包含匹配，就是data参数里的数据，是否只要包含文本框里的数据就显示
				autoFill: false,    //自动填充
				dataType:'json',
				extraParams: {format: 'json' },
				parse: function(data) {  
		            return $.map(data, function(row) {  
		                return {  
		                    data: row,  
		                    value: row.kiName,  
		                    result: row.kiName 
		                }  
		            });  
		        },  
				formatItem: function(row, i, max) {
					return   row.kiName ;
				},
				formatMatch: function(row, i, max) {
					return row.kiName;
				},
				formatResult: function(row) {
					return row.kiName;
				}
			})
		});

###使用中后代接受传值总是乱码，最后发现编码格式是iso8859-1 于是问题得到解决，希望能帮助到你们**

	String str = new String(q.getBytes("iso8859-1"),"UTF-8");//q为插件默认的参数名，为前台传过来的值 
	     
###如果你对autocomplete.js 中的 q: lastWord(term),  做过url编码，后台用url解码也能解决乱码问题

后台传数据的话要指定编码否则显示的数据会是乱码

	@RequestMapping("/selectKidAutoComplete")
		public String selectKidAutoComplete(String q,HttpServletRequest request,HttpServletResponse response) {
	        //注意这里q是默认的传值名称，不能用其他代替，不管你页面<input/>标签中id和name写的是什么，这里统一用q接收值
			response.setContentType("application/json;charset=UTF-8");     
			response.setCharacterEncoding("UTF-8");     
			String str = null;
			try {
				if (q != null && !"".equals(q)) {
					str = new String(q.getBytes("ISO8859-1"),"UTF-8");
				}
			} catch (UnsupportedEncodingException e1) {
				e1.printStackTrace();
			}
			List<MedKids> procDefList = new ArrayList<MedKids>();
			if (!"".equals(str) && str != null ) {
				procDefList = medKidsService.selectKidAutoComplete(str);
			}
			PrintWriter out;
			try {
				out = response.getWriter();
				out.print(JSON.toJSONString(procDefList).toString());
				out.flush();
				out.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
			return null;
		}

最后找了一点资料
>* minChars (Number):    
    在触发autoComplete前用户至少需要输入的字符数.Default: 1，如果设为0，在输入框内双击或者删除输入框内内容时显示列表
* width (Number):    
    指定下拉框的宽度. Default: input元素的宽度
* max (Number):    
    autoComplete下拉显示项目的个数.Default: 10
* delay (Number):    
    击键后激活autoComplete的延迟时间(单位毫秒).Default: 远程为400 本地10
* autoFill (Boolean):    
    要不要在用户选择时自动将用户当前鼠标所在的值填入到input框. Default: false
* mustMatch (Booolean):    
    如果设置为true,autoComplete只会允许匹配的结果出现在输入框,所有当用户输入的是非法字符时将会得不到下拉框.Default: false
* matchContains (Boolean):    
    决定比较时是否要在字符串内部查看匹配,如ba是否与foo bar中的ba匹配.使用缓存时比较重要.不要和autofill混用.Default: false
* selectFirst (Boolean):   
    如果设置成true,在用户键入tab或return键时autoComplete下拉列表的第一个值将被自动选择,尽管它没被手工选中(用键盘或鼠标).当然如果用户选中某个项目,那么就用用户选中的值. Default: true
* cacheLength (Number):
    缓存的长度.即对从[数据库](http://lib.csdn.net/base/14)中取到的结果集要缓存多少条记录.设成1为不缓存.Default: 10
* matchSubset (Boolean):
    autoComplete可不可以使用对服务器查询的缓存,如果缓存对foo的查询结果,那么如果用户输入foo就不需要再进行检索了,直接使用缓存.通常是打开这个选项以减轻服务器的负担以提高性能.只会在缓存长度大于1时有效.Default: true
* matchCase (Boolean):
    比较是否开启大小写敏感开关.使用缓存时比较重要.如果你理解上一个选项,这个也就不难理解,就好比foot要不要到FOO的缓存中去找.Default: false
* multiple (Boolean):
    是否允许输入多个值即多次使用autoComplete以输入多个值. Default: false
* multipleSeparator (String):
    如果是多选时,用来分开各个选择的字符. Default: ","
* scroll (Boolean):
    当结果集大于默认高度时是否使用卷轴显示 Default: true
* scrollHeight (Number):
    自动完成提示的卷轴高度用像素大小表示 Default: 180  
* formatItem (Function):
    为每个要显示的项目使用高级标签.即对结果中的每一行都会调用这个函数,返回值将用LI元素包含显示在下拉列表中. Autocompleter会提供三个参数(row, i, max): 返回的结果数组, 当前处理的行数(即第几个项目,是从1开始的自然数), 当前结果数组元素的个数即项目的个数. Default: none, 表示不指定自定义的处理函数,这样下拉列表中的每一行只包含一个值.
* formatResult (Function):
    和formatItem类似,但可以将将要输入到input文本框内的值进行格式化.同样有三个参数,和formatItem一样.Default: none,表示要么是只有数据,要么是使用formatItem提供的值.
* formatMatch (Function):
    对每一行数据使用此函数格式化需要查询的数据格式. 返回值是给内部搜索算法使用的. 参数值row
* extraParams (Object):
    为后台(一般是服务端的脚本)提供更多的参数.和通常的作法一样是使用一个键值对对象.如果传过去的值是{ bar:4 },将会被autocompleter解析成my_autocomplete_backend.php?q=foo&bar=4 (假设当前用户输入了foo). Default: {}
* result (handler) Returns: jQuery
    此事件会在用户选中某一项后触发，参数为：    event: 事件对象. event.type为result.    data: 选中的数据行.    formatted:formatResult函数返回的值    例如：    $("#singleBirdRemote").result(function(event, data, formatted) {   //如选择后给其他控件赋值，触发别的事件等等   });