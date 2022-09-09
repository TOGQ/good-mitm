# Good Man in the Middle

[![GitHub stars](https://img.shields.io/github/stars/zu1k/good-mitm)](https://github.com/zu1k/good-mitm/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/zu1k/good-mitm)](https://github.com/zu1k/good-mitm/network)
[![Release](https://img.shields.io/github/release/zu1k/good-mitm)](https://github.com/zu1k/good-mitm/releases)
[![GitHub issues](https://img.shields.io/github/issues/zu1k/good-mitm)](https://github.com/zu1k/good-mitm/issues)
[![Build](https://github.com/zu1k/good-mitm/actions/workflows/build-test.yml/badge.svg)](https://github.com/zu1k/good-mitm/actions/workflows/build-test.yml)
[![GitHub license](https://img.shields.io/github/license/zu1k/good-mitm)](https://github.com/zu1k/good-mitm/blob/master/LICENSE)
[![Docs](https://img.shields.io/badge/docs-read-blue.svg?style=flat)](https://docs.mitm.plus)

利用`MITM`技术实现请求和返回的`重写`、`重定向`、`阻断`等操作


不良林 
导航
首页
分类
17科学上网
4网络技术分享
0软件分享
归档
关于
通过在节点服务器部署MITM劫持奈飞cookie实现免费观看netflix
博主： 不良林 发布时间：2022 年 09 月 03 日 722次浏览 2 条评论 6269字数 分类： 网络技术分享
首页正文  


相信大家以前都看过速蛙云充值1元就送奈飞、P站、迪士尼会员的广告，速蛙也算是一线机场了，我测试过他们的线路，速度很不错也很稳定，如果你也用过，那你肯定知道他们还有其它机场没有的福利社，可以免费看奈飞、p站、迪士尼等流媒体，本视频不是广告，因为速蛙因为不可抗拒的原因已经被迫跑路了

本期主要来聊聊他家的福利社，当你在设置中心开启对应服务后，使用他家的节点访问对应的网站，无需会员账号就可以直接免费观看了，这到底是怎么实现的？以及使用这种方式存在什么安全隐患？本期就以奈飞为例带大家来实现同样的功能

详情请看视频

奈飞低价合租：http://api.buliang0.cf/nf
9.3折优惠码：BLL93

教程文档
安装x-ui：

bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)

检测是否解锁奈飞：

#项目地址：https://github.com/sjlleo/netflix-verify

#下载检测解锁程序
wget -O nf https://github.com/sjlleo/netflix-verify/releases/download/v3.1.0/nf_linux_amd64 && chmod +x nf

#执行
./nf
warp解锁奈飞（WireGuard 网络接口模式）
上期教程：奈飞解锁方式大全

# 项目地址：https://p3terx.com/archives/cloudflare-warp-configuration-script.html

# 自动配置 WARP WireGuard 双栈全局网络
bash <(curl -fsSL git.io/warp.sh) d

# 自动配置 WARP WireGuard IPv4 网络
bash <(curl -fsSL git.io/warp.sh) 4

# 自动配置 WARP WireGuard IPv6 网络
bash <(curl -fsSL git.io/warp.sh) 6

# Cloudflare WARP 一键配置脚本 功能菜单
bash <(curl -fsSL git.io/warp.sh) menu
good-mitm

#项目地址：https://github.com/zu1k/good-mitm

# 生成自签CA私钥和证书
./good-mitm genca

# 执行good-mitm
./good-mitm run -r netflix.yaml

# 后台执行
nohup ./good-mitm run -r netflix.yaml > goodmitm.log 2>&1 &
good-mitm配置文件

- name: "netflix"
  mitm: "*.netflix.com"
  filters:
    url-regex: '^https:\/\/(www\.)?netflix\.com'
  actions:
    - modify-request:
        cookie:
          key: NetflixId
          value: 填入你的NetflixId
    - modify-request:
        cookie:
          key: SecureNetflixId
          value: 填入你的SecureNetflixId
    - modify-response:
        cookie:
          key: NetflixId
          remove: true
    - modify-response:
        cookie:
          key: SecureNetflixId
          remove: true
x-ui配置模板

{
  "api": {
    "services": [
      "HandlerService",
      "LoggerService",
      "StatsService"
    ],
    "tag": "api"
  },
  "inbounds": [
    {
        "listen": "127.0.0.1",
        "port": 30000, 
        "protocol": "socks", 
        "sniffing": {
            "enabled": true,
            "destOverride": ["http", "tls"]
        }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
        "tag": "netflix_proxy",
        "protocol": "http",
        "settings": {
         "servers": [
           {
             "address": "127.0.0.1",
             "port": 34567
           }
         ]
        },
        "streamSettings": {
             "security": "none",
             "tlsSettings": {
               "allowInsecure": false
            }
        }
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "policy": {
    "system": {
      "statsInboundDownlink": true,
      "statsInboundUplink": true
    }
  },
  "routing": {
    "rules": [
      {
        "type": "field",
        "outboundTag": "netflix_proxy",
        "domain": [
          "geosite:netflix"
        ]
      },
      {
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api",
        "type": "field"
      },
      {
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "blocked",
        "type": "field"
      },
      {
        "outboundTag": "blocked",
        "protocol": [
          "bittorrent"
        ],
        "type": "field"
      }
    ]
  },
  "stats": {}
}
证书检测工具

https://docs.microsoft.com/zh-cn/sysinternals/downloads/sigcheck
## 使用方法

这里仅介绍最基本的使用流程，具体使用方法和规则请查看[文档](https://docs.mitm.plus)

### 证书准备

由于`MITM`技术的需要，需要你生成并信任自己的根证书

#### 生成根证书

出于安全考虑，请不要随意信任任何陌生人提供的根证书，你需要自己生成属于自己的根证书和私钥

```shell
good-mitm.exe genca
```

上面命令将会生成私钥和证书，文件将存储在`ca`文件夹下

#### 信任证书

你可以将根证书添加到操作系统或者浏览器的信任区中，根据你的需要自行选择

### 代理

启动Good-MITM，指定使用的规则文件或目录

```shell
good-mitm.exe run -r rules
```

在浏览器或操作系统中使用Good-MITM提供的http代理：`http://127.0.0.1:34567`

## License

**Good-MITM** © [zu1k](https://github.com/zu1k), Released under the [MIT](./LICENSE) License.<br>

> Blog [zu1k.com](https://zu1k.com) · GitHub [@zu1k](https://github.com/zu1k) · Twitter [@zu1k_lv](https://twitter.com/zu1k_lv) · Telegram Channel [@peekfun](https://t.me/peekfun)
