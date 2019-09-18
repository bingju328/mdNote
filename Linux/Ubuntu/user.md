## 权限相关

### chown命令

语法：

```shell
chown [-cfhvR] [--help] [--version] user[:group] file...
```

**参数** :

- user : 新的文件拥有者的使用者 ID
- group : 新的文件拥有者的使用者组(group)
- -c : 显示更改的部分的信息
- -f : 忽略错误信息
- -h :修复符号链接
- -v : 显示详细的处理信息
- -R : 处理指定目录以及其子目录下的所有文件
- --help : 显示辅助说明
- --version : 显示版本

实例:

```shell
chown bingju:bingjugroup file1.txt
```

