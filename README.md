# sni_tls_test.sh

用于 **Reality 协议筛选优质 SNI** 的 TLS 握手延迟测试脚本。

原理与 `openssl s_client -connect 域名:443 -servername 域名` 相同：测量到目标站点的 **完整 TCP + TLS 握手耗时**，并发测试后按延迟升序排列，延迟越低、越稳定的大站越适合作为 Reality 的 SNI。可选检测目标是否支持 **TLS 1.3**（Reality 硬性要求）。

## 快速开始

一行命令运行（无需下载保存）：

```bash
bash -c "$(curl -sSL https://raw.githubusercontent.com/harenaNow/SNI-TLS-test/main/sni_tls_test.sh)" -- --tls13
```

或管道方式：

```bash
curl -sSL https://raw.githubusercontent.com/harenaNow/SNI-TLS-test/main/sni_tls_test.sh | bash -s -- --tls13
```

下载后本地运行：

```bash
chmod +x sni_tls_test.sh
./sni_tls_test.sh --tls13
```

## 参数

| 参数 | 说明 | 默认 |
|---|---|---|
| `-t 秒` | 单次 TLS 握手超时秒数 | `1` |
| `-c N` | 并发测试数 | `8` |
| `-n N` | 只显示最快的 N 个结果 | 全部显示 |
| `-f 文件` | 从文件读取域名（提供时替代内置列表） | 内置列表 |
| `--tls13` | 同时检测 TLS 1.3 支持 | 关闭 |
| `-h, --help` | 显示帮助 | |
| `-V, --version` | 显示版本 | |

域名也可直接写在参数后面：

```bash
./sni_tls_test.sh --tls13 -n 10 www.nvidia.com amd.com www.icloud.com
```

## 输出示例

```text
SNI TLS 握手延迟测试 v1.0.0
openssl: OpenSSL 3.0.13 | 计时: bash 内置 EPOCHREALTIME | 超时工具: GNU timeout
超时: 1s | 并发: 8 | 域名数: 47 (已去重)

#     DOMAIN                                            LATENCY  TLS1.3
1     amd.com                                             47 ms  yes
2     www.icloud.com                                      54 ms  yes
3     www.nvidia.com                                      54 ms  yes
-     192.0.2.1                                         TIMEOUT
-     nonexistent.invalid                                  FAIL

完成: 共 5 个域名, 成功 3, 失败/超时 2
最快: amd.com (47 ms)
Reality 推荐 (最快且支持 TLS1.3): amd.com (47 ms)
```

字段说明：

- **LATENCY**：TCP 连接 + TLS 握手的总耗时（毫秒），越低越好
- **TLS1.3**：`yes` 支持（Reality 可用）/ `no` 不支持 / `n/a` 未检测
- **TIMEOUT**：连接或握手超时（常见于无路由、被墙、IPv6 不通的域名）
- **FAIL**：快速失败（常见于 DNS 解析失败、端口不通）
- **Reality 推荐**：开启 `--tls13` 时，自动选出「延迟最低且支持 TLS 1.3」的域名

## 自定义域名列表

使用 `-f` 指定文件（每行一个域名，`#` 开头为注释），以下写法均可，自动去重：

```text
# 大厂常用站
www.nvidia.com
https://www.apple.com/
cdn.jsdelivr.net:443
WWW.Microsoft.COM
```

## 选择 SNI 的建议

1. 优先选 **延迟最低** 且 `TLS1.3=yes` 的结果
2. 选知名的、有全球 CDN 的大站（未指向前端机所在地区的更好）
3. 避免选择已被墙或与你 VPS 同机房的域名
4. 对候选域名可加测多次取稳定值：`./sni_tls_test.sh -t 2 域名`

## 兼容性

- 依赖：`bash openssl awk sed grep sort`，无其他依赖
- 已适配：Debian 11/12、Ubuntu、CentOS 等 GNU 系发行版；自动探测 bash 内置 `EPOCHREALTIME` / GNU `date %3N` 计时，GNU / BusyBox `timeout`
- Alpine：默认无 bash，先执行 `apk add bash openssl`
- macOS 未适配（BSD date / 无 timeout）
