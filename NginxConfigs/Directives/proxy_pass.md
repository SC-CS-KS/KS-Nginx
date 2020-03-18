# proxy_pass



## 注意

1. 在proxy_pass前面用了rewrite，这种情况下，proxy_pass是无效的
2. location里是正则表达式，这种情况下，proxy_pass里最好不要有URI