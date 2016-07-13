# sqlinj-ant

##一、程序说明

分布式、全覆盖、半自动化 sql注入扫描器

解决问题：传统扫描器对post请求的无力，对登录后请求的无力，扫描效率低，覆盖效果差

通过使用代理服务器主动提交的方式收集全部请求保存到redis中，可覆盖全部http接口
python调用sqlmapapi遍历上一步收集的url进行测试，通过多节点部署sqlmap极大提高识别效率


##二、文件说明
.
|____autoinj.py 			调用sqlmapapi提供的api进行sql注入测试 对存在注入漏洞的请求返回请求信息
|____console.py 			主程序  获取用户参数调用autoinj进行注入测试
|____param.py 				参数配置
|____preprocess.py 			数据预处理程序 将redis中hash保存的请求信息以set形式保存到 待测试 key中
|____proxy.conf 			nginx代理配置 记录http请求到redis

##md三、使用说明
1.环境要求
python 2.7
redis
openresty 版本别太低
sqlmap

2.配置准备
a.安装redis
b.安装openresty并添加proxy.conf 配置http代理服务器 修改proxy.conf中redis地址，确保redis的连接
c.浏览器使用代理，访问web系统，redis自动记录全部请求
 ---数据搜集end---
d.修改param.py中redis连接方式  
e.在扫描节点部署sqlmap并启动sqlmapapi的服务（多节点提高速度）
f.调用preprocess.py 将要扫描的站点转移到db1下set中(原始请求数据保留在db0，db1作任务队列)
g.通过console.py调用sqlmapapi进行扫描
  python console.py http://node1.com:8775
  python console.py http://node2.com:8775
  python console.py http://node2.com:8775
  扫描结果将保存到db1中 key=sql.inj
 ---扫描end---

 如提示缺少module，请自行安装

 四、其它
 redis数据保存说明

 资源数据-HASH（全部http请求）
 key 		proxy.hostname
 field 		url
 value 		method host url args remote 等 （json格式）

待扫描数据-SET
key 		proxy.set
value 		资源数据中value的集合

扫描结果-HASH
key 		sqlinj
field 		注入点url
value 		注入请求详情



