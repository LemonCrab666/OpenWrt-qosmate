# QoSmate: Quality of Service for OpenWrt

基于 [@hudra0](https://github.com/hudra0/qosmate) 做出的汉化

QoSmate 是一个针对 OpenWrt 路由器的服务质量（QoS）解决方案，旨在优化网络性能，同时允许对特定流量类型进行受控优先级排序。它使用 nftables 进行数据包分类，并提供 CAKE（通用应用增强保持）和 HFSC（分层公平服务曲线）队列管理机制来管理流量。它使用 tc-ctinfo 在入站流量中恢复 DSCP 标记。

该项目建立在 [@dlakelan](https://github.com/dlakelan) 的卓越工作基础上，并扩展了他的 [SimpleHFSCgamerscript](https://github.com/dlakelan/routerperf/blob/master/SimpleHFSCgamerscript.sh)的功能，增加了用户友好的界面。QoSmate 融合了各种 QoS 系统的概念，包括 SQM、DSCPCLASSIFY 和 cake-qos-simple，以提供全面的流量控制方案。

QoSmate 的关键特点包括：
- 支持 HFSC 和 CAKE 队列管理机制
- 基于 LuCI 的界面，便于配置
- 通过 CLI 和 UI 提供 DSCP 标记和流量优先级选项
- 自动安装和设置软件包

虽然 QoSmate 可以改善各种网络流量，包括游戏和其他对延迟敏感的应用，但只有在正确配置时才能提高整体网络性能。

重要提示：有效的 QoS 关注于战略性优先级，而不是所有流量的统一提升。QoSmate 允许您优先考虑特定流量类型，但必须谨慎使用这一功能。过度优先级可能会抵消 QoS 的好处，因为提升过多流量本质上等同于没有优先级。请记住，给予优待的每个数据包，其他数据包可能会经历更高的延迟或丢失。目标是创建一个平衡、高效的网络环境，而不是优先考虑所有内容。

## 1. 安装

在安装 QoSmate 之前，请确保：

1. 禁用并停止任何现有的 QoS 服务或脚本（例如 SQM、Qosify、DSCPCLASSIFY、SimpleHFSCgamerscript 等），以避免冲突。
2. 重启路由器，以清除旧设置，确保干净的启动。

### a) 后端安装

使用以下命令安装 QoSmate 后端（包含主脚本/初始化脚本/热插拔和配置文件）：

```bash
wget -O /etc/init.d/qosmate https://raw.githubusercontent.com/hudra0/qosmate/main/etc/init.d/qosmate && chmod +x /etc/init.d/qosmate && wget -O /etc/qosmate.sh https://raw.githubusercontent.com/hudra0/qosmate/main/etc/qosmate.sh && chmod +x /etc/qosmate.sh && wget -O /etc/config/qosmate https://raw.githubusercontent.com/hudra0/qosmate/main/etc/config/qosmate
```

### b) 前端安装

使用以下命令安装 [luci-app-qosmate](https://github.com/hudra0/luci-app-qosmate) :

```bash
mkdir -p /www/luci-static/resources/view/qosmate /usr/share/luci/menu.d /usr/share/rpcd/acl.d /usr/libexec/rpcd && \
wget -O /www/luci-static/resources/view/qosmate/settings.js https://raw.githubusercontent.com/hudra0/luci-app-qosmate/main/htdocs/luci-static/resources/view/settings.js && \
wget -O /www/luci-static/resources/view/qosmate/hfsc.js https://raw.githubusercontent.com/hudra0/luci-app-qosmate/main/htdocs/luci-static/resources/view/hfsc.js && \
wget -O /www/luci-static/resources/view/qosmate/cake.js https://raw.githubusercontent.com/hudra0/luci-app-qosmate/main/htdocs/luci-static/resources/view/cake.js && \
wget -O /www/luci-static/resources/view/qosmate/advanced.js https://raw.githubusercontent.com/hudra0/luci-app-qosmate/main/htdocs/luci-static/resources/view/advanced.js && \
wget -O /www/luci-static/resources/view/qosmate/rules.js https://raw.githubusercontent.com/hudra0/luci-app-qosmate/main/htdocs/luci-static/resources/view/rules.js && \
wget -O /www/luci-static/resources/view/qosmate/connections.js https://raw.githubusercontent.com/hudra0/luci-app-qosmate/main/htdocs/luci-static/resources/view/connections.js && \
wget -O /www/luci-static/resources/view/qosmate/custom_rules.js https://raw.githubusercontent.com/hudra0/luci-app-qosmate/main/htdocs/luci-static/resources/view/custom_rules.js && \
wget -O /usr/share/luci/menu.d/luci-app-qosmate.json https://raw.githubusercontent.com/hudra0/luci-app-qosmate/main/root/usr/share/luci/menu.d/luci-app-qosmate.json && \
wget -O /usr/share/rpcd/acl.d/luci-app-qosmate.json https://raw.githubusercontent.com/hudra0/luci-app-qosmate/main/root/usr/share/rpcd/acl.d/luci-app-qosmate.json && \
wget -O /usr/libexec/rpcd/luci.qosmate https://raw.githubusercontent.com/hudra0/luci-app-qosmate/main/root/usr/libexec/rpcd/luci.qosmate && \
chmod +x /usr/libexec/rpcd/luci.qosmate && \
/etc/init.d/rpcd restart && \
/etc/init.d/uhttpd restart

```

### c) 使用

1. 安装完成后，启动 QoSmate 服务：
```
/etc/init.d/qosmate start
```
1. 访问 LuCI 网页界面，导航到 网络 > QoSmate。
2. 配置基本设置：对于基本配置，调整以下关键参数：
    - **WAN 接口**: 选择您的 WAN 接口
    - **下载速率 (kbps)**: 设置为实际下载速度的 80-90%
    - **上传速率 (kbps)**: 设置为实际上传速度的 80-90%
    - **根队列管理机制**: 在 HFSC（默认）和 CAKE 之间选择
3. 应用更改

#### 自动设置功能

对于喜欢自动配置的用户，QoSmate 提供自动设置功能：

1. 在 QoSmate 设置页面，点击“开始自动设置”
2. 可选，输入您的游戏设备的 IP 地址以进行优先级设置
3. 等待速度测试和配置完成

**注意**: 基于路由器的速度测试可能会低估您的实际连接速度。为了更精确的设置，从 LAN 设备运行速度测试并手动输入结果。自动设置提供了一个有用的起点，但可能需要手动微调以获得最佳性能。
## 2. QoSmate Configuration Settings

### Basic and Global Settings

| Config option | Description                                                                                                                                                                                                                    | Type              | Default |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | ------- |
| enabled       | Enables or disables QoSmate. Set to 1 to enable, 0 to disable.                                                                                                             | boolean           | 1       |
| WAN           | Specifies the WAN interface. This is crucial for applying QoS rules to the correct network interface. It's typically the interface connected to your ISP.                                                                      | string            | eth1    |
| DOWNRATE      | Download rate in kbps. Set this to about 80-90% of your actual download speed to allow for overhead and prevent bufferbloat. This creates a buffer that helps maintain low latency even when the connection is fully utilized. | integer           | 90000   |
| UPRATE        | Upload rate in kbps. Set this to about 80-90% of your actual upload speed for the same reasons as DOWNRATE.                                                                                                                    | integer           | 45000   |
| ROOT_QDISC    | Specifies the root queueing discipline. Options are 'hfsc' or 'cake'                                                                                                                                                           | enum (hfsc, cake) | hfsc    |

### HFSC Specific Settings

| Config option       | Description                                                                                                                                                                                                                                                                                          | Type                                         | Default                 |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | ----------------------- |
| LINKTYPE            | Specifies the link type. This affects how overhead is calculated. 'ethernet' is common for most connections, 'atm' for DSL, and 'DOCSIS' for cable internet.                                                                                                                                         | enum (ethernet, atm, DOCSIS)                 | ethernet                |
| OH                  | Overhead in bytes. This accounts for layer 2 encapsulation overhead. Adjust based on your connection type.                                                                                                                                                                                           | integer                                      |                         |
| gameqdisc           | Queueing discipline for game traffic. Options include 'pfifo ' 'bfifo' , 'fq_codel' , 'red' , and 'netem'. Each has different characteristics for managing realtime traffic.                                                                                                                         | enum (pfifo, bfifo fq_codel, red, netem)     | pfifo                   |
| GAMEUP              | Upload bandwidth reserved for gaming in kbps. Formula ensures minimum bandwidth for games even on slower connections.                                                                                                                                                                                | integer                                      | (UPRATE*15/100+400)     |
| GAMEDOWN            | Download bandwidth reserved for gaming in kbps. Similar to GAMEUP, but for download traffic.                                                                                                                                                                                                         | integer                                      | (DOWNRATE*15/100+400)   |
| nongameqdisc        | Queueing discipline for non-realtime traffic. 'fq_codel' or 'cake'.                                                                                                                                                                                                                                  | enum (fq_codel, cake)                        | fq_codel                |
| nongameqdiscoptions | Additional cake options when cake is set as the non-game qdisc.                                                                                                                                                                                                                                      | string                                       | "besteffort ack-filter" |
| MAXDEL              | Maximum delay in milliseconds. This sets an upper bound on queueing delay, helping to maintain responsiveness even under load.                                                                                                                                                                       | integer                                      | 24                      |
| PFIFOMIN            | Minimum number of packets in the pfifo queue.                                                                                                                                                                                                                                                        | integer                                      | 5                       |
| PACKETSIZE          | Pfifo average packet size in bytes. Used in calculations for queue limits. Adjust if you know your game traffic has a significantly different average packet size.                                                                                                                                   | integer                                      | 450                     |
| netemdelayms        | Artificial delay added by netem in milliseconds, only used if 'gameqdisc' is set to 'netem'. This is useful for testing or simulating higher latency connections. Netem applies the delay in both directions, so if you set a delay of 10 ms, you will experience a total of 20 ms cumulative delay. | integer                                      | 30                      |
| netemjitterms       | Jitter added by netem in milliseconds. Simulates network variability, useful for testing how applications handle inconsistent latency.                                                                                                                                                               | integer                                      | 7                       |
| netemdist           | Distribution of delay for netem. Options affect how the artificial delay is applied, simulating different network conditions.                                                                                                                                                                        | enum (uniform, normal, pareto, paretonormal) | normal                  |
| pktlossp            | Packet loss percentage for netem. Simulates network packet loss, useful for testing application resilience.                                                                                                                                                                                          | string                                       | none                    |

### CAKE Specific Settings
All cake settings are described in the tc-cake man.

| Config option            | Description                                                                                                                                                    | Type                                       | Default   |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ | --------- |
| COMMON_LINK_PRESETS      | Preset for common link types. Affects overhead calculations and default behaviors. 'ethernet' or 'conservative' is suitable for most connections.              | enum (ethernet, docsis, ... see cake man.) | ethernet  |
| OVERHEAD                 | Manual overhead setting. If set, overrides the preset. Useful for fine-tuning or unusual setups.                                                               | integer                                    |           |
| MPU                      | Minimum packet unit size. If set, overrides the preset.                                                                                                        | integer                                    |           |
| LINK_COMPENSATION        | Additional compensation for link peculiarities.                                                                                                                | string (atm, ptm, noatm)                   |           |
| ETHER_VLAN_KEYWORD       | Keyword for Ethernet VLAN compensation. Used when VLAN tagging affects packet sizes.                                                                           | string                                     |           |
| PRIORITY_QUEUE_INGRESS   | Priority queue handling for incoming traffic. 'diffserv4' uses 4 tiers of priority based on DSCP markings.                                                     | enum (diffserv3, diffserv4, diffserv8)     | diffserv4 |
| PRIORITY_QUEUE_EGRESS    | Priority queue handling for outgoing traffic. Usually matched with INGRESS for consistency.                                                                    | enum (diffserv3, diffserv4, diffserv8)     | diffserv4 |
| HOST_ISOLATION           | Enables host isolation in CAKE. Prevents one client from monopolizing bandwidth, ensuring fair distribution among network devices. (dual-srchost/dual-dsthost) | boolean                                    | 1         |
| NAT_INGRESS              | Enables NAT lookup for incoming traffic. Important for correct flow identification in NAT scenarios.                                                           | boolean                                    | 1         |
| NAT_EGRESS               | Enables NAT lookup for outgoing traffic.                                                                                                                       | boolean                                    | 1         |
| ACK_FILTER_EGRESS        | Controls ACK filtering. 'auto' enables it when download/upload ratio ≥ 15, helping to prevent ACK floods on asymmetric connections.                            | enum (auto, 1, 0)                          | auto      |
| RTT                      | Round Trip Time estimation. If set, used to optimize CAKE's behavior for your specific network latency.                                                        | integer                                    |           |
| AUTORATE_INGRESS         | Enables CAKE's automatic rate limiting for ingress. Can adapt to changing network conditions but may be less predictable.                                      | boolean                                    | 0         |
| EXTRA_PARAMETERS_INGRESS | Additional parameters for ingress CAKE qdisc. For advanced tuning, allows passing custom options directly to CAKE.                                             | string                                     |           |
| EXTRA_PARAMETERS_EGRESS  | Additional parameters for egress CAKE qdisc. Similar to INGRESS, but for outgoing traffic.                                                                     | string                                     |           |

### Advanced Settings

| Config option          | Description                                                                                                                                                           | Type    | Default            |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ------------------ |
| PRESERVE_CONFIG_FILES  | If enabled, configuration files are preserved during system upgrades. Ensures your QoS settings survive firmware updates.                                             | boolean | 1                  |
| WASHDSCPUP             | Sets DSCP to CS0 for outgoing packets after classification                                                                                                            | boolean | 1                  |
| WASHDSCPDOWN           | Sets DSCP to CS0 for incoming packets before classification                                                                                                           | boolean | 1                  |
| BWMAXRATIO             | Maximum ratio between download and upload bandwidth. Prevents ACK floods on highly asymmetric connections by limiting download speed relative to upload.              | integer | 20                 |
| ACKRATE                | Sets rate limit for TCP ACKs, helps prevent ACK flooding / set to 0 to disable ACK rate limit                                                                         | integer | (UPRATE * 5 / 100) |
| UDP_RATE_LIMIT_ENABLED | Downgrades UDP traffic exceeding 450 pps to lower priority                                                                                                            | boolean | 1                  |
| UDPBULKPORT            | UDP ports for bulk traffic. Often used for torrent traffic. Helps identify and manage high-bandwidth, lower-priority traffic.                                         | string  |                    |
| TCPBULKPORT            | TCP ports for bulk traffic. Comma-separated list or ranges. Similar to UDPBULKPORT, but for TCP-based bulk transfers.                                                 | string  |                    |
| VIDCONFPORTS           | [Legacy - use rules] Ports used for video conferencing and other high priority traffic. Uses the Fast Non-Realtime (1:12) queue.                                      | string  |                    |
| REALTIME4              | [Legacy - use rules] IPv4 addresses of devices to receive real-time priority (Only UDP). Useful for gaming consoles or VoIP devices that need consistent low latency. | string  |                    |
| REALTIME6              | [Legacy - use rules] IPv6 addresses for real-time priority. Equivalent to REALTIME4 but for IPv6 networks.                                                            | string  |                    |
| LOWPRIOLAN4            | [Legacy - use rules] IPv4 addresses of devices to receive low priority. Useful for limiting impact of bandwidth-heavy but non-time-sensitive devices.                 | string  |                    |
| LOWPRIOLAN6            | [Legacy - use rules] IPv6 addresses for low priority. Equivalent to LOWPRIOLAN4 but for IPv6 networks.                                                                | string  |                    |

### DSCP Marking Rules

QoSmate allows you to define custom DSCP (Differentiated Services Code Point) marking rules to prioritize specific types of traffic. These rules are defined in the `/etc/config/qosmate` file under the `rule` sections and via luci-app-qosmate.

| Config option | Description                                                                                            | Type                                                                                                                      | Default |
| ------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- | ------- |
| name          | A unique name for the rule. Used for identification and logging.                                       | string                                                                                                                    |         |
| proto         | The protocol to match. Determines which type of traffic the rule applies to.                           | enum (tcp, udp, icmp)                                                                                                     |         |
| src_ip        | Source IP address or range to match. Can use CIDR notation for networks.                               | string                                                                                                                    |         |
| src_port      | Source port or range to match. Can use individual ports or ranges like '1000-2000'.                    | string                                                                                                                    |         |
| dest_ip       | Destination IP address or range to match. Similar to src_ip in format.                                 | string                                                                                                                    |         |
| dest_port     | Destination port or range to match. Similar to src_port in format.                                     | string                                                                                                                    |         |
| class         | DSCP class to assign to matching packets. Determines how the traffic is prioritized in the QoS system. | enum (cs0, cs1, cs2, cs3, cs4, cs5, cs6, cs7, af11, af12, af13, af21, af22, af23, af31, af32, af33, af41, af42, af43, ef) |         |
| counter       | Enable packet counting for this rule. Useful for monitoring and debugging.                             | boolean                                                                                                                   | 0       |

#### Example rule configuration:
```
config rule 
	option name 'gaming_traffic' 
	option proto 'udp' 
	option src_ip '192.168.1.100' 
	option dest_port '3074' 
	option class 'cs5' 
	option counter '1'
```
This rule would mark UDP traffic from IP 192.168.1.100 to port 3074 with the CS5 DSCP class, which is typically used for gaming traffic, and enable packet counting for this rule.

#### Additional DSCP Marking Rule Examples

1. Prioritizing Video Conferencing Traffic:
```
config rule
    option name 'zoom_traffic'
    option proto 'tcp udp'
    list dest_port '3478-3479'
    list dest_port '8801-8802'
    option class 'af41'
    option counter '1'
```
Explanation: This rule marks both TCP and UDP traffic to typical Zoom ports with the DSCP class AF41. Using `list` for `dest_port` allows specifying multiple port ranges. AF41 is well-suited for interactive video conferencing as it provides high priority without impacting the highest priority traffic.

2. Low Priority for Peer-to-Peer Traffic:
```
config rule
    option name 'p2p_traffic'
    option proto 'tcp udp'
    list src_port '6881-6889'
    list dest_port '6881-6889'
    option class 'cs1'
    option counter '1'
```
Explanation: This rule assigns low priority to P2P traffic (like BitTorrent) by marking it as CS1. Using `list` for both `src_port` and `dest_port` covers both incoming and outgoing P2P traffic.

3. Call of Duty Game Traffic:
```
config rule
    option name 'cod1'
    option proto 'udp'
    option src_ip '192.168.1.208'
    option src_port '3074'
    option dest_port '30000-65535'
    option class 'cs5'
    option counter '1'

config rule
    option name 'cod2'
    option proto 'udp'
    option dest_ip '192.168.1.208'
    option dest_port '3074'
    option class 'cs5'
    option counter '1'
```
Explanation: These rules prioritize Call of Duty game traffic. The first rule targets outgoing traffic from the game console (IP 192.168.1.208), while the second rule handles incoming traffic. Both use CS5 class, which is typically used for gaming traffic due to its high priority. The wide range of destination ports in the first rule covers the game's server ports.

4. Generic Game Console/Gaming PC Traffic:
```
config rule
    option name 'Game_Console_Outbound'
    option proto 'udp'
    option src_ip '192.168.1.208'
    list dest_port '!=80'
    list dest_port '!=443'
    option class 'cs5'
    option counter '1'

config rule
    option name 'Game_Console_Inbound'
    option proto 'udp'
    option dest_ip '192.168.1.208'
    list src_port '!=80'
    list src_port '!=443'
    option class 'cs5'
    option counter '1'
```
Explanation: These rules provide a more generic approach to prioritizing game console/gaming pc traffic. The outbound rule prioritizes all UDP traffic from the console (192.168.1.208) except for ports 80 and 443 (common web traffic). The inbound rule does the same for incoming traffic. This approach ensures that game-related traffic gets priority while allowing normal web browsing to use default priorities. The use of '!=' (not equal) in the port lists demonstrates how to exclude specific ports from the rule.
This is more or less equivalent to the `realtime4` and `realtime6` variables from the SimpleHFSCgamer script. However, this rule is even better as it excludes UDP port 80 and 443, which are often used for QUIC. This is likely less of an issue on a gaming console than on a gaming PC, where a YouTube video using QUIC might be running alongside the game.

This rule is also applied when the auto-setup is used via CLI or UI and a Gaming Device IP (optional) is entered.

## Command Line Interface
QoSmate can be controlled and configured via the command line. The basic syntax is:
```
/etc/init.d/qosmate [command]
```
```
Available commands:
        start           Start the service
        stop            Stop the service
        restart         Restart the service
        reload          Reload configuration files (or restart if service does not implement reload)
        enable          Enable service autostart
        disable         Disable service autostart
        enabled         Check if service is started on boot
        running         Check if service is running
        status          Service status
        trace           Start with syscall trace
        info            Dump procd service info
        check_version   Check for updates
        update          Update qosmate
        auto_setup      Automatically configure qosmate
        expand_config   Expand the configuration with all possible options
        auto_setup_noninteractive   Automatically configure qosmate with no interaction
```
### Update QoSmate
```
/etc/init.d/qosmate update
```

## Troubleshooting
If you encounter issues with the script or want to verify that it's working correctly, follow these steps:

1. Disable DSCP washing (egress and ingress)
2. Update and install tcpdump: `opkg update && opkg install tcpdump`
3. Mark an ICMP ping to a reliable destination (e.g., 1.1.1.1) with a specific DSCP value using a DSCP Marking Rules.
4. Ping the destination from your LAN client.
5. Use `tcpdump -i <your wan interface> -v -n -Q out icmp` to display outgoing traffic (upload) and verify that the TOS value is not 0x0. Make sure to **set the right interface (WAN Interface).** 
6. Use `tcpdump -i ifb-<your wan interface> -v -n icmp` to display incoming traffic (download) and verify that the TOS value is not 0x0. Make sure to **set the right interface (ifb + <yourwaninterface)**
7. Install watch: `opkg update && opkg install procps-ng-watch`
8. Check traffic control queues:

   - When using HFSC as root qdisc:
     ```
     watch -n 2 'tc -s qdisc | grep -A 2 "parent 1:11"'
     ```
     Replace "1:11" with the desired class. The packet count should increase with the ping in both directions.

   - When using CAKE as root qdisc:
     ```
     watch -n 1 'tc -s qdisc show | grep -A 20 -B 2 "diffserv4"'
     ```

   The output will show you if packets are landing in the correct queue.

WIP...
(Include additional troubleshooting steps, such as how to verify if QoSmate is working correctly, common issues and their solutions, etc.)

## Uninstallation

To remove QoSmate from your OpenWrt router:

1. Stop and disable the QoSmate service:
```
/etc/init.d/qosmate stop
```
2. Remove the QoSmate files:
```
rm /etc/init.d/qosmate /etc/qosmate.sh /etc/config/qosmate
```
3. Remove the LuCI frontend files:
```
rm -r /www/luci-static/resources/view/qosmate
rm /usr/share/luci/menu.d/luci-app-qosmate.json
rm /usr/share/rpcd/acl.d/luci-app-qosmate.json
```
4. vRestart the rpcd and uhttpd services:
```
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
```
5. Reboot your router to clear any remaining settings.

## Contributing

Contributions to QoSmate are welcome! Please submit issues and pull requests on the GitHub repository.

## Acknowledgements

QoSmate is inspired by and builds upon the work of SimpleHFSCgamerscript, SQM, cake-qos-simple, qosify and DSCPCLASSIFY. I thank all contributors and the OpenWrt community for their valuable insights and contributions.
