# rewrite

```nginx
	rewrite_log
		rewrite_log on|off；   # 繁忙的服务器中不建议打开
		是否将重写过程记录在错误日志中，默认为notice级别；默认为off
	return
		用于结束rewrite规则，并且为客户返回状态码：可以使用的状态码有204，400，402-406，500-504等
		server {
    ...
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 last;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  break;
    return  403;
    ...
}
```
rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。

## 作用域
server{},location{},if{}
只能对域名后边的除去传递的参数外的字符串起作用
例如 http://seanlook.com/a/we/index.php?id=1&u=str
只对/a/we/index.php重写

## 语法
rewrite regex replacement [flag];
* flag
```text
last ：相当于Apache里德(L)标记，表示完成rewrite；  
break；本条规则匹配完成后，终止匹配，不再匹配后面的规则  
redirect：返回302临时重定向，浏览器地址会显示跳转后的URL地址  
permanent：返回301永久重定向，浏览器地址栏会显示跳转后的URL地址  
	    last和break用来实现URL重写，浏览器地址栏URL地址不变  
```
* rewrite 和 location
rewrite 是在同一域名内更改获取资源的路径
location 是对一类路径做控制访问或反向代理，可以proxy_pass到其他机器。

很多情况下rewrite也会写在location里，它们的执行顺序是：
```text
	执行server块的rewrite指令
	执行location匹配
	执行选定的location中的rewrite指令
	如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件
	循环超过10次，则返回500 Internal Server Error错误。
```
