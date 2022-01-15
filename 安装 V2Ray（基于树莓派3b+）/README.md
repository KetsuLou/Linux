# 安装 V2Ray（基于树莓派3b+）  
[本文全部文件地址-树莓派3b+安装V2ray.rar](https://wwe.lanzoup.com/iY0qNysv8ab)  
  
### 安装V2ray  
安装 V2Ray。可以使用 V2Ray 提供的 go.sh 脚本安装，由于 GFW 会恶化对 GitHub 的访问，直接运行脚本几乎无法安装，建议先从 [v2ray-core/releases](https://github.com/v2ray/v2ray-core/releases) 将安装包`v2ray-linux-arm.zip`下载到树莓派, 使用`--local`参数从本地安装  
![本地文件](https://github.com/408029164/Linux/raw/main/安装V2Ray（基于树莓派3b+）/pic/本地文件.png)  
![树莓派文件](https://img-blog.csdnimg.cn/35633a415af14618afb7f2b7ee101be6.png)  
```Bash
sudo bash ./install-release.sh --local v2ray-linux-arm32-v7a.zip
```  
注意终端的运行目录，安装之后是这样的  
![安装之后](https://img-blog.csdnimg.cn/52cb8a35ceaa423c888ce854093ccb88.png)  
有几个文件目录需要记一下（可能目录会不一样，我只以我的为准）  
```
/usr/local/bin：v2ray可执行文件
/usr/local/etc/v2ray：config.json
/usr/local/share/v2ray：geoip.dat和geosite.dat
```  
![v2ray可执行文件](https://img-blog.csdnimg.cn/1406c2cd26d741bc9bf5c85885d9fe7e.png)  
![config.json](https://img-blog.csdnimg.cn/3c0162bb2f69417eaf3eba5af234392f.png)  
![geoip.dat和geosite.dat](https://img-blog.csdnimg.cn/d913284f212f47aba81c29104a51b495.png)  
之后可能会用到的命令  
```Bash
/usr/local/bin/v2ray -test -config /usr/local/etc/v2ray/config.json
#查看配置文件是否出错
sudo systemctl start v2ray # 启动v2ray服务
sudo systemctl stop v2ray # 停止v2ray服务
sudo systemctl status v2ray # 查看v2ray运行状态
sudo systemctl enable v2ray # 将v2ray加入开机自启动
```  
![查看配置文件是否出错](https://img-blog.csdnimg.cn/071127728138464c8c3f73f1523f39a6.png)  
执行 `curl -so /dev/null -w "%{http_code}" google.com -x http://127.0.0.1:10808`确认V2Ray 已经可以科学上网(命令中`http`指`inbound`协议为`http`，`10808`指该`inbound`端口是`10808`)。如果执行这个命令出现了`301`或`200`这类数字的话代表可以科学上网，如果长时间没反应或者是`000`的话说明不可以科学上网。  
![执行curl](https://img-blog.csdnimg.cn/3ab237e652db497bbcd6f57004d780d2.png)  
原帖：  
![原帖](https://img-blog.csdnimg.cn/c3729299b611414eb3de07911d14f159.png)  
### 国内直连  
将geoip.dat与geosite.dat移动到目标文件夹（原来的建议备份）  
![geoip.dat和geosite.dat](https://img-blog.csdnimg.cn/d913284f212f47aba81c29104a51b495.png)  
![备份后](https://img-blog.csdnimg.cn/5d807ca7deb643a3887df3eaf9cd7bf2.png)  
  
### 配置 V2Ray  
通过 v2rayN 可以导出节点配置为客户端配置，以下为 我个人的WS+TLS 的配置文件示例，已配置国内直连（请勿直接使用以下配置）：  
```JSON
{
  "dns": {
    "hosts": {
      "domain:googleapis.cn": "googleapis.com"
    },
    "servers": [
      "1.1.1.1",
      {
        "address": "223.5.5.5",
        "domains": [
          "geosite:cn"
        ],
        "expectIPs": [
          "geoip:cn"
        ],
        "port": 53
      }
    ]
  },
  "inbounds": [
    {
      "port": 10808,
      "protocol": "http",//原来是用socks我修改成http了
      "settings": {
        "auth": "noauth",
        "udp": true,
        "userLevel": 8
      },
      "sniffing": {
        "destOverride": [
          "http",
          "tls"
        ],
        "enabled": false
      },
      "tag": "socks"
    },
    {
      "port": 10809,
      "protocol": "http",
      "settings": {
        "userLevel": 8
      },
      "tag": "http"
    },
    {
      "listen": "127.0.0.1",
      "port": 10853,
      "protocol": "dokodemo-door",
      "settings": {
        "address": "1.1.1.1",
        "network": "tcp,udp",
        "port": 53
      },
      "tag": "dns-in"
    }
  ],
  "log": {
    "loglevel": "warning"
  },
  "outbounds": [
    {
      "mux": {
        "concurrency": -1,
        "enabled": false
      },
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "usa-washington.lvuft.com",
            "port": 443,
            "users": [
              {
                "alterId": 4,
                "id": "aba50dd4-5484-3b05-b14a-4661caf862d5",
                "level": 8,
                "security": "auto"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "tlsSettings": {
          "allowInsecure": false,
          "serverName": "usa-washington.lvuft.com"
        },
        "wsSettings": {
          "headers": {
            "Host": "usa-washington.lvuft.com"
          },
          "path": "/ws"
        }
      },
      "tag": "proxy"
    },
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "settings": {
        "response": {
          "type": "http"
        }
      },
      "tag": "block"
    },
    {
      "protocol": "dns",
      "tag": "dns-out"
    }
  ],
  "policy": {
    "levels": {
      "8": {
        "connIdle": 300,
        "downlinkOnly": 1,
        "handshake": 4,
        "uplinkOnly": 1
      }
    },
    "system": {
      "statsOutboundUplink": true,
      "statsOutboundDownlink": true
    }
  },
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      {
        "inboundTag": [
          "dns-in"
        ],
        "outboundTag": "dns-out",
        "type": "field"
      },
      {
        "ip": [
          "1.1.1.1"
        ],
        "outboundTag": "proxy",
        "port": "53",
        "type": "field"
      },
      {
        "ip": [
          "223.5.5.5"
        ],
        "outboundTag": "direct",
        "port": "53",
        "type": "field"
      },
      {
        "domain": [
          "domain:googleapis.cn"
        ],
        "outboundTag": "proxy",
        "type": "field"
      },
      {
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "direct",
        "type": "field"
      },
      {
        "ip": [
          "geoip:cn"
        ],
        "outboundTag": "direct",
        "type": "field"
      },
      {
        "domain": [
          "geosite:cn"
        ],
        "outboundTag": "direct",
        "type": "field"
      }
    ]
  },
  "stats": {}
}
```  
将之保存为`config.json`并测试是否可用  
```
/usr/local/bin/v2ray -test -config /usr/local/etc/v2ray/config.json #查看配置文件是否出错
```  
![保存为config.json](https://img-blog.csdnimg.cn/6067d16036b14bd18aa6d51c01b246e7.png)  
  
### 修改系统代理  
终端输入  
```
sudo vim /etc/profile
```  
在末尾添加如下代码：  
```
# proxy
export http_proxy=127.0.0.1:10808
export https_proxy=127.0.0.1:10808
#端口根据之前config.json配置文件里面的inbounds写
```  
![修改系统代理](https://img-blog.csdnimg.cn/59fd4024b8534932a94bfd1d449154a9.png)  
然后`Esc`，输入`:wq`保存退出，重启v2ray  
### 常见问题  
##### 1、	官网的一键安装v2ray脚本无法进行安装报错 #2685  
![常见问题1](https://img-blog.csdnimg.cn/3aee298044c24f77a46e1f37ed3da26b.png)  
```
curl -Ls https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh | sudo bash
```  
（目测用我教程的安装方法不会出现这样的问题）  
##### 2、	客户端提示 Socks: unknown Socks version:  
可能原因：客户端配置的 inboud 设置成了 socks 而浏览器的代理协议设置为 http。  
修正方法：修改配置文件使客户端的 inboud 的 protocol 和浏览器代理设置的协议保持一致。  
##### 3、【Linux】Uubntu18.04 apt-get update报错：Unsupported proxy configured: 127.0.0.1://8118  
[【Linux】Uubntu18.04 apt-get update报错：Unsupported proxy configured: 127.0.0.1://8118](https://blog.csdn.net/qq408029164/article/details/122508744)  
