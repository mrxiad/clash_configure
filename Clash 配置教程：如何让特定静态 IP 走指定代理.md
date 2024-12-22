# Clash 配置教程：如何让特定静态 IP 走指定代理

在使用 **Clash** 进行网络代理配置时，有时你可能需要将特定的静态 IP 地址的流量通过特定的代理节点转发。本文将详细指导你如何在 Clash 配置文件中设置，使得访问特定静态 IP 时，流量自动通过预设的代理节点。

## 目录

1. [前提条件](#前提条件)
2. [配置概述](#配置概述)
3. 步骤详解
   - [1. 定义专用 Proxy Group](#1-定义专用-proxy-group)
   - [2. 添加静态 IP 规则](#2-添加静态-ip-规则)
   - [3. 验证配置](#3-验证配置)
4. [完整配置示例](#完整配置示例)
5. [测试配置](#测试配置)
6. [常见问题与解决方案](#常见问题与解决方案)
7. [总结](#总结)

------

## 前提条件

- **Clash 已安装并正常运行**：确保你已经安装了 Clash 客户端，并且可以正常连接代理服务器。
- **基本的 YAML 语法知识**：Clash 的配置文件使用 YAML 格式，了解基本的缩进和语法规则。
- **代理节点信息**：已知你要使用的代理节点（如 Trojan、Socks5 等）的详细信息。

## 配置概述

在 Clash 中，主要有三个部分需要理解：

- **proxies**：定义所有可用的代理节点。
- **proxy-groups**：将多个代理节点分组，并定义选择策略。
- **rules**：根据特定条件（如域名、IP 地址）决定流量走向哪个代理或代理组。

我们的目标是：

1. **创建一个专用的 Proxy Group**，包含你希望用于访问特定静态 IP 的代理节点。
2. **添加一条规则**，将目标为该静态 IP 的流量指向刚创建的 Proxy Group。

## 步骤详解

### 1. 定义专用 Proxy Group

首先，我们需要在 `proxy-groups` 部分定义一个新的代理分组，用于处理访问特定静态 IP 的流量。

```yaml
proxy-groups:
  - name: ProxyForMyStaticIP
    type: select
    proxies:
      - '🇭🇰|香港-中转 01'
      - '🇭🇰|香港-中转 02'
      - '🇺🇸|美国-中转 01'
```

**解释：**

- **name**：为新分组命名，例如 `ProxyForMyStaticIP`。
- **type**：选择 `select` 类型，允许你手动选择一个代理节点。
- **proxies**：列出你希望用于该分组的代理节点名称。确保这些节点已经在 `proxies` 部分定义。

> **提示**：你可以根据需要添加更多备用代理节点，以增强连接的稳定性和容错性。

### 2. 添加静态 IP 规则

接下来，在 `rules` 部分添加一条规则，将目标为特定静态 IP 的流量指向刚才创建的 Proxy Group。

```yaml
rules:
  - IP-CIDR,45.203.148.236/32,ProxyForMyStaticIP
  # 其他规则...
  - GEOIP,CN,DIRECT
  - MATCH,Proxy
```

**解释：**

- IP-CIDR

  ：匹配 IP 地址范围。

  - `45.203.148.236/32`：匹配单个 IP 地址 `45.203.148.236`。
  - `ProxyForMyStaticIP`：指定流量应通过的 Proxy Group 名称。

- **规则顺序**：确保此规则在其他相关规则之前（如 `GEOIP,CN,DIRECT`）以优先匹配。

### 3. 验证配置

配置完成后，务必验证 YAML 语法的正确性以及 Clash 配置的有效性。

#### 验证 YAML 语法

可以使用在线工具如 [YAML Lint](https://www.yamllint.com/) 来检查配置文件的语法是否正确。

#### 验证 Clash 配置

1. **加载配置文件**：在 Clash 客户端中加载更新后的配置文件。
2. **检查日志**：查看 Clash 日志，确保没有语法错误或配置问题。
3. **连接测试**：尝试访问特定静态 IP，确认流量是否通过指定的代理节点转发。

------

## 完整配置示例

以下是一个集成了上述步骤的完整配置示例。请根据你的实际情况调整代理节点名称和 IP 地址。

```yaml
mixed-port: 7890
allow-lan: true
bind-address: '*'
mode: rule
log-level: info
external-controller: '127.0.0.1:9090'
dns:
  enable: true
  ipv6: false
  default-nameserver: [223.5.5.5, 119.29.29.29]
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  use-hosts: true
  nameserver: ['https://doh.pub/dns-query', 'https://dns.alidns.com/dns-query']
  fallback: ['https://doh.dns.sb/dns-query', 'https://dns.cloudflare.com/dns-query', 'https://dns.twnic.tw/dns-query', 'tls://8.8.4.4:853']
  fallback-filter: { geoip: true, ipcidr: [240.0.0.0/4, 0.0.0.0/32] }

proxies:
  - { name: '🇭🇰|香港-中转 01', type: trojan, server: CN-FS.catcat-123.com, port: 9120, password: f27aca06-2553-4dab-8c8a-a8e15aea510f, udp: true, sni: hkcmi-01.oneok.link }
  - { name: '🇭🇰|香港-中转 02', type: trojan, server: CN-SHCM.catcat-123.com, port: 9130, password: f27aca06-2553-4dab-8c8a-a8e15aea510f, udp: true, sni: hkt-02.oneok.link }
  - { name: '🇺🇸|美国-中转 01', type: trojan, server: CN-SHCU.catcat-123.com, port: 8300, password: f27aca06-2553-4dab-8c8a-a8e15aea510f, udp: true, sni: us-01.oneok.link }
  - { name: 'S5_USA_Paypal', type: socks5, server: 45.203.148.236, port: 6505, username: *****, password: **** }
  # ... 其他代理节点

proxy-groups:
  - name: ProxyForMyStaticIP
    type: select
    proxies:
      - '🇭🇰|香港-中转 01'
      - '🇭🇰|香港-中转 02'
      - '🇺🇸|美国-中转 01'

  - name: 节点选择
    type: select
    proxies:
      - 自动选择
      - '剩余流量：8.89 GB'
      - '距离下次重置剩余：16 天'
      - 套餐到期：2025-01-05
      - '🇭🇰|香港-中转 01'
      - '🇭🇰|香港-中转 02'
      - '🇭🇰|香港-中转 03'
      - # ... 其他代理节点

rules:
  - IP-CIDR,45.203.148.236/32,ProxyForMyStaticIP
  - DOMAIN,api.sakuracat.club,DIRECT
  - DOMAIN-SUFFIX,gcld-line.com,节点选择
  - # ... 其他规则
  - GEOIP,CN,DIRECT
  - MATCH,Proxy
```

**注意事项：**

- **确保 Proxy Group 名称一致**：`ProxyForMyStaticIP` 在 `rules` 和 `proxy-groups` 中名称必须一致。
- **代理节点名称正确**：在 `proxies` 和 `proxy-groups` 中引用的代理节点名称必须完全匹配，包括大小写和特殊字符。
- **规则顺序**：静态 IP 的规则应放在更广泛的规则之前，以确保优先匹配。

------

## 测试配置

1. **加载配置**：在 Clash 客户端中加载或更新配置文件。
2. **查看日志**：打开 Clash 的日志界面，确保没有语法错误或代理连接失败的提示。
3. **访问静态 IP**：使用浏览器或其他网络工具访问 `45.203.148.236`，并监控日志确认流量是否通过 `ProxyForMyStaticIP` 组的代理节点转发。
4. **检查 IP 路径**：你可以使用 [IP 路径检测工具](https://www.iplocation.net/) 来确认访问静态 IP 时的实际 IP 路径，确保流量通过预期的代理节点。

------

## 常见问题与解决方案

### 1. 配置文件加载失败

- **症状**：Clash 无法启动，显示 YAML 解析错误。

- 解决方案

  ：

  - 检查 YAML 缩进是否正确（使用两个空格）。
  - 确保所有键值对之间使用冒号 `:` 分隔。
  - 使用在线工具如 [YAML Lint](https://www.yamllint.com/) 验证配置文件的语法。

### 2. 流量未通过指定代理

- **症状**：访问静态 IP 时，流量未通过 `ProxyForMyStaticIP` 组的代理节点。

- 解决方案

  ：

  - 确认规则顺序是否正确，静态 IP 规则应在更广泛的规则之前。
  - 确认 `ProxyForMyStaticIP` 组中包含有效且可用的代理节点。
  - 检查代理节点的连接状态，确保代理节点正常工作。

### 3. 代理节点连接失败

- **症状**：指定的代理节点无法连接，导致访问静态 IP 失败。

- 解决方案

  ：

  - 检查代理节点的服务器地址、端口、密码等信息是否正确。
  - 确认代理服务器是否在线，或者联系服务提供商获取帮助。
  - 尝试使用 `Proxy Group` 中的其他备用代理节点。

------

## 总结

通过本文的指导，你应该能够成功配置 Clash，使得访问特定的静态 IP 地址时，流量自动通过指定的代理节点转发。这种配置在需要特定流量走特定代理的场景下非常有用，如访问被封锁的服务、优化网络路径等。

**关键步骤回顾：**

1. **定义专用 Proxy Group**：包含希望用于转发静态 IP 流量的代理节点。
2. **添加静态 IP 规则**：将目标静态 IP 的流量指向该 Proxy Group。
3. **验证与测试**：确保配置正确，代理节点正常工作。

希望这篇教程对你有所帮助，祝你网络配置顺利！