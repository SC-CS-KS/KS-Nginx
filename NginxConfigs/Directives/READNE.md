
```nginx
	日志
			log_format
			access_log
			error_log
		由ngx_http_log_module实现
	client_body_buffer_size
		If the request body size is more than the buffer size, then the entire (or partial) request body is written into a temporary file.
	try_files
		
			语法
				try_files file ... uri 或 try_files file ... = code
			作用域
				server location
			配合命名location，可以部分替代原本常用的rewrite配置方式，提高解析效率
			作用是按顺序检查文件是否存在，返回第一个找到的文件或文件夹(结尾加斜线表示为文件夹)，如果所有的文件或文件夹都找不到，会进行一个内部重定向到最后一个参数。
	root & alias
		
			root和alias都可以定义在location模块中，都是用来指定请求资源的真实路径，比如：
				location /i/ {
    root /data/w3;
}
			请求 http://foofish.net/i/top.gif 这个地址时，那么在服务器里面对应的真正的资源是 /data/w3/i/top.gif文件，注意 真实的路径是root指定的值加上location指定的值 。
			而 alias 正如其名，alias指定的路径是location的别名，不管location的值怎么写，资源的 真实路径都是 alias 指定的路径 ，比如：
				location /i/ {
    alias /data/w3/;
}
			同样请求 http://foofish.net/i/top.gif 时，在服务器查找的资源路径是：/data/w3/top.gif
		其他区别：
			alias 只能作用在location中，而root可以存在server、http和location中。
			alias 后面必须要用 “/” 结束，否则会找不到文件，而 root 则对 ”/” 可有可无。
	server_tokens
		
			Syntax:	server_tokens on | off | string;
			Default:	
				server_tokens on;
			Context:	http, server, location
			Enables or disables emitting nginx version in error messages and in the “Server” response header field.
			Additionally, as part of our commercial subscription, starting from version 1.9.13 the signature in error messages and the “Server” response header field value can be set explicitly using the string with variables. An empty string disables the emission of the “Server” field.
	proxy_set_header
		
			允许重新定义或者添加发往后端服务器的请求头。
			语法:	proxy_set_header field value;
				value可以包含文本、变量或者它们的组合。 
				 当且仅当当前配置级别中没有定义proxy_set_header指令时，会从上面的级别继承配置。
			默认值:	
				默认情况下，只有两个请求头会被重新定义：
					proxy_set_header Host $proxy_host;
					proxy_set_header Connection close;
			上下文:	http, server, location
			自定义
				proxy_set_header test paroxy_test;
			应用
				想要在web端获得用户的真实ip
					经过反向代理后，由于在客户端和web服务器之间增加了中间层，因此web服务器无法直接拿到客户端的ip，通过$remote_addr变量拿到的将是反向代理服务器的ip地址
					proxy_set_header            X-real-ip $remote_addr;
					proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
						有做X-Forwarded-For设置的话,每次经过proxy转发都会有记录,格式就是client1, proxy1, proxy2,以逗号隔开各个地址
	underscores_in_headers
		
			如果想header支持下划线的话，需要增加如下配置
				是否允许在header的字段中带下划线
			语法：underscores_in_headers on|off
			默认值：off
			使用字段：http, server
```
```nginx
extend
	echo_exec
		第三方模块 ngx_echo 
```