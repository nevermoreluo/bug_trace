# bug_trace
收集记录bug






## Wsl

### <3>init: (28848) ERROR: UtilConnectToInteropServer:300: connect failed 2
环境：`Docker Desktop`, `wsl2`

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
