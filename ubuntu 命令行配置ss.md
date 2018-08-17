## 1. download ss client
```shell
git clone https://github.com/ssrbackup/shadowsocksr
```
## 2. edit config 

    edit the profile  ./config.json  
    
```json
{
    "server": "0.0.0.0",    //server ip 
    "server_ipv6": "::",    
    "server_port": 80890,   //server port
    "local_address": "127.0.0.1",
    "local_port": 1080, 

    "password": " ",
    "method": "chacha20",   //encrypt method
    "protocol": "auth_sha1_v4", // protol
    "protocol_param": "", 
    "obfs": "http_simple",   //mix
    "obfs_param": "",          // mix parameter
    "speed_limit_per_con": 0,
    "speed_limit_per_user": 0,

    "additional_ports" : {}, // only works under multi-user mode
    "additional_ports_only" : false, // only works under multi-user mode
    "timeout": 120,
    "udp_timeout": 60,
    "dns_ipv6": false,
    "connect_verbose_info": 0,
    "redirect": "",
    "fast_open": false
}
```
    
## 3. run shadowsocksr
```shell
python ~/shadowsocksr/shadowsocks/local/py -c ~/shadowsocksr/config.json
```
## 4. configurating local agent (http ,https)
   shadowsocks use Protocol Socks5, use Polipo to configurating http or https
   
```shell   
apt-get install polipo
vim /etc/polipo/config
```
   replace by follow content:
   
```go
# This file only needs to list configuration variables that deviate
# from the default values. See /usr/share/doc/polipo/examples/config.sample
# and "polipo -v" for variables you can tweak and further information.
logSyslog = false
logFile = "/var/log/polipo/polipo.log"

socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5

chunkHighMark = 50331648
objectHighMark = 16384

serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32

proxyAddress = "0.0.0.0"
proxyPort = 8123
```
   
    restart Polipo
 ```shell
/etc/init.d/polipo restart 
```
    
    configurating proxy
```shell
export http_proxy="http://127.0.0.1:8123/"
export https_proxy="http://127.0.0.1:8123"
```



ENV:   ubuntu 16.04.3 LTS

TOOLS: shadowsocksr


