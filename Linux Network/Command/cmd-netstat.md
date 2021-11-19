### netstat
- `netstat -tulnp` 查看端口情况
- `netstat -atn |grep 192.168.205.11` 查看是否有到 192.168.205.11 的连接

- `-t`: display only tcp connections
- `-n`: show numerical addresses
- `-u`: display only udp connections
- `-l`: show only listening sockets
- `-p`: show name of process IDs, name of programs
- `-r`: kernel routing table
- `-a`: display all sockets (default: connected)