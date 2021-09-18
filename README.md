# bug_trace
工作好几年，才发现原来你遇到的bug其实都和你有缘  
记录下bug，哪怕先记录一些流水账式的bug场景，也给自己一些记忆中留下点检索的面包屑  
收集记录bug&error






## Wsl

### getsocketopt error: 92 Protocol not available
环境： `Wsl`  
原理：  
wsl在版本1，没有TCP_INFO的实现，即不论getsocketopt或者setsocketopt不允许在wsl1中调用关于TCP_INFO的调用, 例如
```
getsockopt (m_Handle, IPPROTO_TCP, TCP_INFO, &info, (socklen_t *)&optlen)
```

`注意: KEEPALIVE同样存在问题`

按官方的说法是wsl2里面已经修复了该问题

[参考wsl issues#1982](https://github.com/microsoft/WSL/issues/1982)


解决方式：  
升级wsl到wsl2
然后在wsl中执行python测试
`python3 -c "import socket; socket.socket().getsockopt(socket.SOL_TCP, socket.TCP_INFO)"`如果没有报下面的报错就说明修复了
```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python2.7/socket.py", line 228, in meth
    return getattr(self._sock,name)(*args)
socket.error: [Errno 92] Protocol not available
```



### <3>init: (28848) ERROR: UtilConnectToInteropServer:300: connect failed 2
环境：`Docker Desktop`, `Wsl2`  
原理：  
还不太明白 [参考wsl issues#5065](https://github.com/microsoft/WSL/issues/5065)  

解决方式：  
在Docker+wsl2环境下执行docker-compose中遇到错误，将一下脚本写入~/.bashrc 文件最下方，
```bash
fix_wsl2_interop() {
    for i in $(pstree -np -s $$ | grep -o -E '[0-9]+'); do
        if [[ -e "/run/WSL/${i}_interop" ]]; then
            export WSL_INTEROP=/run/WSL/${i}_interop
        fi
    done
}
```
另外开启一个终端或者重新打开当前终端，或者通过命令强制刷新`source ~/.bashrc`
后执行 `fix_wsl2_interop` 即可修复


#### CMakeFile Permission denied
环境： `Wsl2`, `Clion`  
原理：  
Clion 默认启动时 使用wsl中的root用户创建了文件，导致权限不正常,删除对应文件 重新启动机器即可恢复

解决方式：  
查看项目下.idea文件为root权限，则进行以下操作  
```
cd <ProjectDir>  
sudo rm -r .idea cmake-build-*  
```
关机重启查看文件权限,恢复到当前用户即可正常运行


### test.sh: line 3: $'\r': command not found
环境： `linux`, `shell`  
原理：   
在windows换下编辑文件时，部分ide会将换行转换  

解决方式：  
替换掉\r即可 `sed -i 's/\r$//' test.sh` 


### rsync error: failed to set times on "/foo/bar": Operation not permitted
环境： `linux`, `rsync`  
原理：
同步目录时间存在权限问题，通过跳过同步目录时间选项可以解决，[stackoverflow](https://stackoverflow.com/questions/667992/rsync-error-failed-to-set-times-on-foo-bar-operation-not-permitted)

解决方式：  
跳过同步目录时间，添加--omit-dir-times命令到rsync参数  



## Nodejs

### getaddrinfo ENOTFOUND xxx.com
环境： `nodejs`  
原理：  
本质原因是，nodejs在发起http请求时无法解析域名，通常来说检查本地dns，以及相应域名的解析是否正确

解决方式：  
排查dns解析  
1.通过nslookup或dig查看解析是否正常  
2.查看本机hosts文件是否存在异常改动  
3.查看本机是否存在代理服务，并劫持了53端口解析  




