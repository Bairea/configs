终端启用代理
```PowerShell
# v2rayN用的10809端口
$env:HTTP_PROXY="http://127.0.0.1:10809"
$env:HTTPS_PROXY="http://127.0.0.1:10809"
$env:ALL_PROXY="http://127.0.0.1:10809"
$env:WS_PROXY="http://127.0.0.1:10809"
$env:WSS_PROXY="http://127.0.0.1:10809"

codex
```
