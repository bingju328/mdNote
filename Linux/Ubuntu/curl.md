GET
```
curl localhost:9999/api/daizhige/article -v
```
-v 查看请求的详细参数

```
curl localhost:8091/index/component/save -X POST -H "Content-Type:application/json" -d '{}'
```
-d  == --data
我们可以用 -X PUT 和 -X DELETE 来指定另外的请求方法


curl POST 上传文件
传输文件，其实这个对于 curl 来说，也是小菜一碟。

我们用 -F "file=@__FILE_PATH__" 的请示，传输文件即可。命令如下：
```
curl localhost:8000/api/v1/upimg -F "file=@/Users/fungleo/Downloads/401.png" -H "token: 222" -v
```
