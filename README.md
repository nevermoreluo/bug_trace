# bug_trace
工作好几年，才发现原来你遇到的bug其实都和你有缘  
记录下bug，
收集记录bug&error






## Wsl

### <3>init: (28848) ERROR: UtilConnectToInteropServer:300: connect failed 2
环境：`Docker Desktop`, `wsl2`  
原理： 还不太明白  

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

[参考wsl issues#5065](https://github.com/microsoft/WSL/issues/5065)


### test.sh: line 3: $'\r': command not found
环境： `linux shell`  
原理： 在windows换下编辑文件时，部分ide会将换行转换  

解决方式：  
替换掉\r即可 `sed -i 's/\r$//' test.sh` 


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
