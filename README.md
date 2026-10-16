# LiteSpeed主机：为什么快、怎么选，年付十几美元的 VPS 自己搭一套跑 WordPress

先说结论：LiteSpeed 主机，指的是用 LiteSpeed Web Server（或它的开源版 OpenLiteSpeed）做底层 Web 服务器的网站主机。同样跑 WordPress，它的事件驱动架构加上内置的 LSCache 缓存，在速度和并发上普遍优于传统 Apache 主机，HTTP/3 支持也更早进入生产可用状态。

真正的问题是：这东西怎么获得、要花多少钱、免费版和付费版差在哪。网上大部分文章讲完概念就停了，这篇往下多走一步，顺带给一个具体的省钱方案：用 RackNerd 一台年付十几美元的 KVM VPS，自己装免费的 OpenLiteSpeed，跑 WordPress 完全够用。

> **选购速览**：愿意动手的，直接看 RackNerd 的年付促销 KVM VPS，装 CyberPanel（自带 OpenLiteSpeed，零授权费）。多数人选 **2.5GB / 年付 $24.89** 这档就够；预算卡死选 1GB / $10.99；跑 WooCommerce 或带多个站上 4GB / $38.99。👉 [查看 RackNerd 年付促销套餐](https://bit.ly/my-rn)
>
> 完全不想碰命令行的话，这套方案不适合你，直接买预装 LiteSpeed 企业版的托管主机更省心，代价是每月多付一笔授权溢价。

## LiteSpeed 到底是什么？比 Apache 快在哪

Web 服务器是托管环境里负责接收 HTTP 请求、返回网页文件的那个软件，它处理请求的方式直接决定网站速度。LiteSpeed Web Server 由 LiteSpeed Technologies 公司在 2003 年推出，和 Apache、Nginx 属于同一类东西，技术路线更激进。

三种主流服务器的核心差异在这张表里：

|  | Apache | Nginx | LiteSpeed / OpenLiteSpeed |
| --- | --- | --- | --- |
| 架构 | 进程驱动，每个连接开一个进程或线程 | 事件驱动 | 事件驱动 |
| 缓存 | 依赖第三方模块 | 内置 FastCGI 缓存 | 内置 LSCache，支持 ESI 细粒度缓存 |
| PHP 处理 | FastCGI | FastCGI | 自研 LSAPI 接口 |
| HTTP/3 | 靠附加模块，成熟度一般 | 长期处于开发状态 | 生产可用 |

Apache 的进程驱动模式是几十年前流量规模下的产物，应付高并发效率不高；Nginx 和 LiteSpeed 都是事件驱动，用一个进程处理大量连接，内存和 CPU 占用低得多。LiteSpeed 的额外优势在于 LSCache 是服务器级内置的，WordPress 装上官方 LiteSpeed Cache 插件后可以直接吃到底层缓存，不用再叠一层第三方缓存方案。官方给出的基准数据是服务器负载通常减半、TTFB（首字节时间）提升约 3 倍，实际幅度取决于站点本身。

规模上它是后来者：按 W3Techs 的统计，全球接近 12% 的网站跑在 LiteSpeed 上，Apache 加 Nginx 合计接近七成。不过它家的开源 HTTP/3 库和配套的 QUIC.cloud CDN，这两年在 WordPress 加速圈里存在感明显变强。

## 用上 LiteSpeed 的两条路：买托管主机，还是自己搭

两条路对应两种花钱方式：一是买预装 LiteSpeed 企业版的托管主机，省心但贵；二是买一台便宜 KVM VPS，自己装免费的 OpenLiteSpeed，省钱但要动手。

先说授权这件事。LiteSpeed 官方的企业版按 Free Starter、Site Owner、Web Host 三类授权出售，按域名数和服务器规模收费。托管商把企业版预装进套餐，这部分成本最终都摊在你的月费里。OpenLiteSpeed 则是开源版，同一拨人开发、同一套事件驱动架构，免费下载随便部署；代价是企业版里的一部分反 DDoS 工具被砍掉，缓存引擎里最先进的机制也没有，出问题只能靠社区。

算一笔账就明白差别了。RackNerd 的 1GB 促销 KVM VPS 年付 $10.99，折合每月不到 1 美元，软件层零授权费；同样的预算在托管型 LiteSpeed 主机那里，通常只够买最入门的共享套餐，配置和权限还受限。当然，托管主机换来的是完全不用管服务器，这笔溢价买的是省心，值不值取决于你自己。

想走自建路线的，👉 [先看看 RackNerd 当前的年付特价](https://bit.ly/my-rn)，下面会说怎么挑档位。

## 为什么用 RackNerd 的 KVM VPS 来搭

RackNerd 是一家美国低价 VPS 商，靠常年挂出的年付促销 KVM 套餐在海外低价圈里混出了口碑。选它搭 LiteSpeed 的核心原因是 KVM 虚拟化：完整 root 权限，装面板、调内核参数、挂自定义 ISO 都不受限，这正是跑 OpenLiteSpeed 需要的环境。

几件和日常使用相关的事：

- 机房覆盖洛杉矶、圣何塞、西雅图、达拉斯、芝加哥、纽约、亚特兰大等多个美国城市，部分套餐还有阿姆斯特丹可选，下单时在下拉菜单里挑
- 1Gbps 端口，标配独立 IPv4
- SolusVM 控制面板带 VNC 控制台，装系统、重装都自己动手
- 系统镜像覆盖 AlmaLinux、Ubuntu、Debian 等主流发行版，也支持挂自定义 ISO
- 非托管服务：硬件、网络、虚拟化层的问题走 24/7 工单找他们；机器里装什么、怎么优化、安全加固都是自己的事
- 付款支持 PayPal 和国际信用卡

非托管这点要提前想清楚。遇到“面板打不开”“数据库连不上”这类问题时，没人替你修。好处是价格确实压到了底。

## RackNerd 促销套餐怎么选

选档位的原则看两个数：站点数量和内存。下面是它长期挂出的四档年付促销 KVM VPS，全部适合跑 OpenLiteSpeed：

| 套餐 | vCPU | SSD 存储 | 月流量 | 年付价 | 下单 |
| --- | --- | --- | --- | --- | --- |
| 1GB 入门 | 1 核 | 20 GB | 1 TB | $10.99 | [ 年付 $10.99 开通](https://bit.ly/my-rn) |
| 2GB 进阶 | 2 核 | 30 GB | 3 TB | $21.99 | [ 年付 $21.99 开通](https://bit.ly/my-rn) |
| 2.5GB 均衡 | 2 核 | 45 GB | 3.5 TB | $24.89 | [ 年付 $24.89 开通](https://bit.ly/my-rn) |
| 4GB 富余 | 2 核 | 65 GB | 5 TB | $38.99 | [ 年付 $38.99 开通](https://bit.ly/my-rn) |

几点补充。促销价会随批次调整，黑五、新年这类节点经常出现更低的限时档位，下单前以官网 specials 页面实时显示为准；高于 4GB 的配置也有，但跑小型 WordPress 站用不上。促销套餐续费按同一价格执行，通过对应入口下单直接按促销价结算，个别活动才需要优惠码，下单前看清页面说明即可。

> 它家的 VPS 不带自动备份。数据丢了没人替你找回来，WordPress 站从第一天起就要安排备份，用插件定时把数据库和上传目录推到外部存储是最省事的做法。

OpenLiteSpeed 本身吃资源不多，事件驱动架构的定位就是用最小内存撑起大量并发，真正的瓶颈一般在 MySQL 和站点数量上。单个内容站 1GB 加个 swap 就能转；两三个站开缓存，2.5GB 是舒服的位置。

## 动手部分：CyberPanel 一键装 OpenLiteSpeed

全程半小时以内，最难的部分只是复制一条命令。CyberPanel 是免费面板，装好自带 OpenLiteSpeed，还集成了 Let's Encrypt 证书和 WordPress 一键部署。

1. VPS 开通后进 SolusVM 装系统，AlmaLinux 或 Ubuntu 22.04 都行
2. SSH 登录，运行 CyberPanel 官方安装脚本：


sh <(curl https://cyberpanel.net/install.sh || wget -O - https://cyberpanel.net/install.sh)


3. 安装过程中选 OpenLiteSpeed（默认选项就是它；想上企业版要另买授权）
4. 装完用 `https://服务器IP:8090` 进面板，系统防火墙记得放行 8090 端口
5. Websites → Create Website，填域名并勾选自动签发 Let's Encrypt 证书
6. 用面板自带的安装器部署 WordPress
7. 进 WordPress 后台装官方的 LiteSpeed Cache 插件并启用，LSCache 走服务器底层接口，配合官方插件效果最完整

1GB 机型建议装完系统先加 2GB swap，给 MySQL 留点余量，这是低内存 VPS 的常规操作。

机器还没开的话，👉 [这里可以直接开通一台年付促销 VPS](https://bit.ly/my-rn)，从装系统到跑起第一个站，一晚上的事。

## OpenLiteSpeed 和企业版差在哪，要不要补差价

同一团队、同一架构，免费版对个人站足够；企业版多出来的主要是托管和安全场景的能力，值不值得补差价看用途。

企业版的定位是 Apache 直接替换件：能直接读取 Apache 配置文件，mod_rewrite、.htaccess、mod_security 全都兼容，官方说 15 分钟内就能零停机完成替换。它还有服务器级 reCAPTCHA 防 Layer-7 攻击、异步处理 ModSecurity 规则、ESI 边缘缓存这些托管场景才用得上的功能。

OpenLiteSpeed 的差异主要在四处：一部分反 DDoS 工具被砍掉；缓存引擎里最先进的机制没有；对 .htaccess 的处理方式不同，规则只在服务器启动或重启时读取一次，改完必须重启才生效，企业版则像 Apache 一样动态读取；出问题没有官方支持，靠社区。

对应到选型：自己的一两个 WordPress 站，免费版完全够。给客户管站、经常从 Apache 环境迁移、或者需要更完整的防攻击能力，企业版授权的开销才有回报。

## 常见问题

**1GB 内存的 VPS 跑 OpenLiteSpeed + WordPress 够用吗？**
纯内容站、日 PV 几千的量级够用。LiteSpeed 的事件驱动架构内存占用低，瓶颈一般在数据库，加 2GB swap 基本稳住；站点增多或流量上来再升 2.5GB 档。

**OpenLiteSpeed 支持 HTTP/3 吗？**
支持，而且是生产可用级别。LiteSpeed 家的 HTTP/3 在主流服务器里属于最早落地的一批，开源实现 LSQUIC 也对外公开，装完面板在站点设置里开启即可。

**RackNerd 是托管主机吗？出了问题谁管？**
非托管。VPS 的硬件故障、网络问题、面板异常走他们的 24/7 工单；网站程序、面板配置、安全防护、数据备份都是你自己的责任。习惯共享托管全包服务的人需要适应这个差别。

**机房怎么选？国内访问快吗？**
RackNerd 的机房都在海外，没有针对中国大陆的优化线路，走国际常规路由，不同地区体验差异不小。下单前先到官网的 Looking Glass 页面测一下目标机房到你本地的延迟和路由再决定，多数人会在圣何塞和洛杉矶之间选。

**促销价续费会涨价吗？**
不会。促销套餐续费按同一价格执行，不搞首年低价、次年翻倍那一套。

## 按你的情况对号入座

- 一个博客或作品集，日 PV 三五千以内：1GB / $10.99，加 2GB swap
- 两三个 WordPress 站，要开 LSCache 做点小流量：2.5GB / $24.89，大多数人停在这档
- WooCommerce 商店、图片多的站、四个以上站点：4GB / $38.99
- 完全不想碰 SSH 和面板：买预装 LiteSpeed 企业版的托管共享主机，为省心付溢价

下单前最后确认三件事：到 Looking Glass 测一下目标机房的延迟；看清当前促销价和续费规则；想好备份方案再上线。确认完就可以 👉 [去开通一台 RackNerd 年付 KVM VPS](https://bit.ly/my-rn)，剩下的装面板、建站，本文都写过了。
