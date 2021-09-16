## curl
### 下载文件
- 单个文件 `curl -O URL`
- 多个文件 `curl -O URL [-O URL...]`
- 失败重传 `curl -C - -O URL`
- 限速 `curl --limit-rate 1m -O URL`

### 只获得头部
- `curl -I URL`
- `curl -I --http2 URL`
- 新版 curl 默认采用 `--http2`
- `-s` silent 隐藏进度条和错误信息

### 跟随重定向
- `curl -L URL`
- 有时候下载链接有重定向 `curl -OL`

### 设置 UA
- `curl -A "Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0" URL`

### 代理
- 代理 `-x host:port`

### request method
- JSON 格式 `curl -d '{"key1":"value1", "key2":"value2"}' -H "Content-Type: application/json" -X POST URL`
- 文件形式 `curl -d "@data.json" -H "Content-Type: application/json" -X POST URL`

### 发送请求到 socket
- `curl -X POST --unix-socket /var/run/docker.sock -d '{"Image":"nginx"}' -H 'Content-Type: application/json' http://localhost/containers/create`