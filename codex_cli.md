终端启用代理
```PowerShell
$env:HTTP_PROXY="http://127.0.0.1:7890"
$env:HTTPS_PROXY="http://127.0.0.1:7890"
$env:ALL_PROXY="http://127.0.0.1:7890"
$env:WS_PROXY="http://127.0.0.1:7890"
$env:WSS_PROXY="http://127.0.0.1:7890"

codex
```
