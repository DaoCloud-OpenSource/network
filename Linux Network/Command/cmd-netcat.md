### netcat
- `nc [options] host port`
- udp: `nc -u host port`
- Port Scanning: `nc -z -v 10.10.8.8 20-80`. The `-z` option will tell nc to only scan for open ports, without sending any data to them and the `-v` option to provide more verbose information.
- `nc -z -v 10.211.55.2 8070-8080 2>&1 |grep succeeded`
- UDP Port Scanning: `nc -z -v -u 10.10.8.8 20-80`