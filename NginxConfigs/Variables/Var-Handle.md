# Var-Handle
```md
	“存取处理程序”
		“取处理程序”（get handler）
			在读取变量时执行的这段特殊代码
			不同的 Nginx 模块一般会为它们的变量准备不同的“存取处理程序”
				从而让这些变量的行为充满魔法。
		“存处理程序”（set handler）
			改写变量时执行的这段特殊代码
		Nginx 变量也是支持绑定“存取处理程序”的
			Nginx 模块在创建变量时，可以选择是否为变量分配存放值的容器，以及是否自己提供与读写操作相对应的“存取处理程序”。
		不是所有的 Nginx 变量都拥有存放值的容器。
			拥有值容器的变量在 Nginx 核心中被称为“被索引的”（indexed）；反之，则被称为“未索引的”（non-indexed）。
		像 $arg_XXX 这样具有无数变种的变量群，是“未索引的”。
			Nginx 根本不会事先就解析好 URL 参数串，而是在用户读取某个 $arg_XXX 变量时，调用其“取处理程序”，即时去扫描 URL 参数串。
			类似地，内建变量 $cookie_XXX 也是通过它的“取处理程序”，即时去扫描 Cookie 请求头中的相关定义的。
	值容器用作缓存
		在多次读取变量的时候，就只需要调用“取处理程序”计算一次。
		 map 配置指令
			标准 ngx_map 模块
			    map $args $foo {
        default     0;
        debug       1;
    }
				用 map 指令定义了用户变量 $foo 与 $args 内建变量之间的映射关系。
					特别地，用数学上的函数记法 y = f(x) 来说，我们的 $args 就是“自变量” x
						而 $foo 则是“因变量” y
					即 $foo 的值是由 $args 的值来决定的
					或者按照书写顺序可以说，我们将 $args 变量的值映射到了 $foo 变量上。
				映射规则：
					default 是一个特殊的匹配条件，即当其他条件都不匹配的时候，这个条件才匹配
						当这个默认条件匹配时，就把“因变量” $foo 映射到值 0.
					第二行的意思是说，如果“自变量” $args 精确匹配了 debug 这个字符串，则把“因变量” $foo 映射到值 1. 
					将这两行合起来，我们就得到如下完整的映射规则：当 $args 的值等于 debug 的时候，$foo 变量的值就是 1，否则 $foo 的值就为 0.
			map 指令其实是一个比较特殊的例子，因为它可以为用户变量注册“取处理程序”，而且用户可以自己定义这个“取处理程序”的计算规则。
				因为缓存的存在，只在请求生命期中的第一次读取中才被执行
			map 指令是在 server 配置块之外，也就是在最外围的 http 配置块中定义的。
		类似 ngx_map 模块，标准的 ngx_geo 等模块也一样使用了变量值的缓存机制。
	“子请求”（subrequest）
		由 Nginx 正在处理的请求在 Nginx 内部发起的一种级联请求。
		“子请求”在外观上很像 HTTP 请求，但实现上却和 HTTP 协议乃至网络通信一点儿关系都没有。
			它是 Nginx 内部的一种抽象调用，目的是为了方便用户把“主请求”的任务分解为多个较小粒度的“内部请求”，并发或串行地访问多个 location 接口，然后由这些 location 接口通力协作，共同完成整个“主请求”。
			当然，“子请求”的概念是相对的，任何一个“子请求”也可以再发起更多的“子子请求”，甚至可以玩递归调用（即自己调用自己）
		当一个请求发起一个“子请求”的时候，按照 Nginx 的术语，习惯把前者称为后者的“父请求”（parent request）。
			值得一提的是，Apache 服务器中其实也有“子请求”的概念，所以来自 Apache 世界的读者对此应当不会感到陌生。
		 echo_location 指令
			ngx_echo 模块
		    location /main {
        echo_location /foo;
        echo_location /bar;
    }
			其执行是按照配置书写的顺序串行处理的
				即只有当 /foo 请求处理完毕之后，才会接着处理 /bar 请求。
		“子请求”方式的通信是在同一个虚拟主机内部进行的
			Nginx 核心在实现“子请求”的时候，就只调用了若干个 C 函数，完全不涉及任何网络或者 UNIX 套接字（socket）通信。
			由此可以看出“子请求”的执行效率是极高的。
		即便是父子请求之间，同名变量一般也不会相互干扰
			 ngx_echo， ngx_lua，以及 ngx_srcache 在内的许多第三方模块都选择了禁用父子请求间的变量共享。
			不幸的是，一些 Nginx 模块发起的“子请求”却会自动共享其“父请求”的变量值容器
				比如第三方模块 ngx_auth_request.
					auth_request 指令会自动忽略“子请求”的响应体，而只检查“子请求”的响应状态码。当状态码是 2XX 的时候，auth_request 指令会忽略“子请求”而让 Nginx 继续处理当前的请求，否则它就会立即中断当前（主）请求的执行，返回相应的出错页
		不幸的是，并非所有的内建变量都作用于当前请求
			少数内建变量只作用于“主请求”，比如由标准模块 ngx_http_core 提供的内建变量 $request_method.
			变量 $request_method 在读取时，总是会得到“主请求”的请求方法，比如 GET、POST 之类
			并不能通过标准的 $request_method 变量取得“子请求”的请求方法。为了达到我们最初的目的，我们需要求助于第三方模块 ngx_echo 提供的内建变量 $echo_request_method
			类似 $request_method，内建变量 $request_uri 一般也返回的是“主请求”未经解析过的 URL，毕竟“子请求”都是在 Nginx 内部发起的，并不存在所谓的“未解析的”原始形式。
	“主请求”（main request）
		由 HTTP 客户端从 Nginx 外部发起的请求
```