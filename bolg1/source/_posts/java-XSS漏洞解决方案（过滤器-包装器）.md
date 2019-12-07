---
title: XSS漏洞解决方案（过滤器-包装器）
date: 2016-09-20 17:59:44
tags: java
---

最近公司网站发现了XSS漏洞，研究了下，分享如下。
<!--more-->
### 一、XSS扫盲
>[跨站脚本攻击](http://baike.baidu.com/view/2633667.htm)(Cross Site Scripting)，为不和[层叠样式表](http://baike.baidu.com/view/728238.htm)(Cascading Style Sheets, [CSS](http://baike.baidu.com/subview/15916/5236733.htm))的缩写混淆，故将跨站脚本攻击缩写为XSS。恶意攻击者往Web页面里插入恶意Script代码，当用户浏览该页之时，嵌入其中Web里面的Script代码会被执行，从而达到恶意攻击用户的特殊目的。

1)恶意用户，在一些公共区域（例如，建议提交表单或消息公共板的输入表单）输入一些文本，这些文本被其它用户看到，但这些文本不仅仅是他们要输入的文本，同时还包括一些可以在客户端执行的脚本。
2)恶意提交这个表单。
3）其他用户看到这个包括恶意脚本的页面并执行，获取用户的cookie等敏感信息。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2007394-043cb8749933d314.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
### 二、解决方案  
#### 1.建立HttpServletRequestWapper的包装类**  
包装类是对用户发送的请求进行包装，把request中包含XSS代码过滤。

	import java.util.Map;
	
	import javax.servlet.http.HttpServletRequest;  
	import javax.servlet.http.HttpServletRequestWrapper;  
	  
	public class XssHttpServletRequestWrapper extends HttpServletRequestWrapper {  
	    HttpServletRequest orgRequest = null;  
	  
	    public XssHttpServletRequestWrapper(HttpServletRequest request) {  
	    	super(request);
	    }  
	    /** 
	     * 覆盖getParameter方法，将参数名和参数值都做xss过滤。
	     * 如果需要获得原始的值，则通过super.getParameterValues(name)来获取
	     * getParameterNames,getParameterValues和getParameterMap也可能需要覆盖 
	     */  
	    @Override  
	    public String getParameter(String name) {  
	        String value = super.getParameter(xssEncode(name));  
	        if (value != null) {  
	            value = xssEncode(value);  
	        }  
	        return value;  
	    }
	    @Override
	    public String[] getParameterValues(String name) {
	    	String[] value = super.getParameterValues(name);
	    	if(value != null){
	    		for (int i = 0; i < value.length; i++) {
					value[i] = xssEncode(value[i]);
				}
	    	}
	    	return value;
	    }
	    @Override
	    public Map getParameterMap() {
	    	// TODO Auto-generated method stub
	    	return super.getParameterMap();
	    }
	
	    /** 
	     * 覆盖getHeader方法，将参数名和参数值都做xss过滤。
	     * 如果需要获得原始的值，则通过super.getHeaders(name)来获取 
	     * getHeaderNames 也可能需要覆盖 
	     */  
	    @Override  
	    public String getHeader(String name) {  
	  
	        String value = super.getHeader(xssEncode(name));  
	        if (value != null) {  
	            value = xssEncode(value);  
	        }  
	        return value;  
	    }  
	  
	    /** 
	     * 将容易引起xss漏洞的半角字符直接替换成全角字符 在保证不删除数据的情况下保存
	     * @param s 
	     * @return 过滤后的值
	     */  
	    private static String xssEncode(String s) {  
	        if (s == null || s.isEmpty()) {  
	            return s;  
	        }  
	        StringBuilder sb = new StringBuilder(s.length() + 16);  
	        for (int i = 0; i < s.length(); i++) {  
	        	char ch = s.charAt(i);
				switch (ch) {
				case '&':
					sb.append("＆");
					break;
				case '<':
					sb.append("《");
					break;
				case '>':
					sb.append("》");
					break;
				case '"':
					sb.append("“");
					break;
				case '\'':
					sb.append("＼");
					break;
				case '/':
					sb.append("／");
					break;
				default:
					sb.append(ch);
				}
	        }  
	        return sb.toString();  
	    }  
	}  

#### 2.Filter过滤器实现对Request的过滤

	import java.io.IOException;
	
	import javax.servlet.Filter;
	import javax.servlet.FilterChain;
	import javax.servlet.FilterConfig;
	import javax.servlet.ServletException;
	import javax.servlet.ServletRequest;
	import javax.servlet.ServletResponse;
	import javax.servlet.http.HttpServletRequest;
	
	import com.lyms.wxyl.base.wrapper.XssHttpServletRequestWrapper;
	
	public class XssFilter implements Filter {
	
		public void destroy() {
			// TODO Auto-generated method stub
		}
		/**
		 * 过滤器用来过滤的方法
		 */
		public void doFilter(ServletRequest request, ServletResponse response,FilterChain chain) throws IOException, ServletException {
			//包装request
			XssHttpServletRequestWrapper xssRequest = new XssHttpServletRequestWrapper((HttpServletRequest) request);
			chain.doFilter(xssRequest, response);
		}
		public void init(FilterConfig filterConfig) throws ServletException {
			// TODO Auto-generated method stub
		}
	}

#### 3.在web.xml中定义好Filter  

	<filter>
		<filter-name>XssFilter</filter-name>
		<filter-class>top.suroot.base.filter.XssFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>XssFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
  
以上是解决办法，打断点进行调试把，你会看到Filter和Wrapper详细的工作流程。