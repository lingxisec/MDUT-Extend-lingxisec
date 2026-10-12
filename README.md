<h1 align="center">MDUT-Extend 二开版</h1>
<h3 align="center">面向授权数据库安全测试的跨平台工具</h3>

> 基于 [MDUT-Extend v1.4.0](https://github.com/DeEpinGh0st/MDUT-Extend-Release/releases/tag/v1.4.0) 二次开发。
>
> By **凌曦安全团队**

---

## 一、项目介绍

**MDUT-Extend 二开版**面向已获得明确授权的数据库渗透测试、安全研究和防御验证。

本版本在保留上游数据库安全验证能力的基础上，增加了每连接 SOCKS5 代理、GUI SQL 查询、代理配置管理和 MCP 接口，并优化了常用窗口布局。HTTP 隧道功能已移除，内网访问统一使用 SOCKS5 代理通道。

> 本项目仅供已获得合法、明确授权的 CTF、靶场、竞赛沙箱、内部测试和安全教育环境使用。

## 二、核心能力

| 能力         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| 多数据库支持 | 支持 MySQL、Microsoft SQL Server、Oracle、PostgreSQL、Redis 和 MongoDB；MongoDB 当前主要提供连接与信息查看能力。 |
| SOCKS5 代理  | 支持按数据库连接选择 SOCKS5 代理，代理配置集中管理，连接保存时采用配置快照。 |
| GUI SQL 查询 | 为 MySQL、Microsoft SQL Server、Oracle 和 PostgreSQL 提供独立 SQL 查询标签页和动态结果展示。 |
| MCP 接口     | 支持 HTTP 和 stdio 两种模式，可进行连接、会话、SQL、文件、插件和代理管理。 |
| Redis 能力   | 集成 DoubleFree、TDigest、命令恢复、SO 加载及计划任务清理等上游 v1.4.0 能力。 |
| 跨平台执行器 | 提供 Windows、Linux 和 macOS ARM 对应的插件及 Redis 执行器。 |

​	相对上游 v1.4.0 的主要改动

- **新增每连接 SOCKS5 代理**：JDBC 直连流量可经本地中继走指定 SOCKS5（如 suo5 / frp 等），连接保存时可选择直连或已保存的代理。
- **新增代理管理**：通过“设置 → 代理管理”统一维护代理，支持新增、修改、删除、端口与重名校验及保存状态提示。
- **新增 GUI SQL 查询**：MySQL、MSSQL、Oracle、PostgreSQL 窗口提供独立查询标签页，支持预置只读语句、自定义 SQL 和动态结果列表；`INSERT`、`UPDATE` 等语句显示影响行数。
- **新增 MCP stdio 模式**：支持 `--mcp` 与统一入口 `MdutLauncher`；HTTP 模式保留 Bearer 强制鉴权，共提供 18 个 MCP 工具。
- **接入 Redis v1.4 能力**：集成 DoubleFree / TDigest、命令恢复文件、SO 加载和计划任务清理；MCP 提供 6 个 DoubleFree 生命周期工具。
- **移除 HTTP 隧道**：内网访问统一使用 SOCKS5 代理通道。
- **界面优化**：调整窗口尺寸与日志区布局，修复 Redis 利用页裁剪问题，启动时显示授权与免责声明。

## 三、运行要求

- GUI 使用带 JavaFX 的 JDK 8。
- 推荐 Oracle JDK 8 或 Zulu FX 8；JDK 11 及以上如果没有 JavaFX，GUI 可能启动失败。
- MCP 使用 JDK 8 即可；无 JavaFX 环境可直接运行 `java -cp mdut.jar mcp.StdioMcpServer`。
- 只有 Redis 相关外置脚本需要 Node.js / Python，属于可选能力。
- 运行目录需要保留 `mdut.jar`、`Driver/`、`Plugins/`、`config.yaml` 和 `data.db`。

启动 GUI：

```bash
start.bat
start.sh
```

## 五、MCP 使用

### 5.1 HTTP 模式

```bash
java -jar mdut.jar \
  --mcp-http \
  --bind 127.0.0.1 \
  --port 5567 \
  --token '你的强随机Token'
```

Token 也可以通过环境变量 `MDUT_MCP_TOKEN` 提供。HTTP 模式强制 Bearer 鉴权；缺少 Token 时拒绝启动。

客户端配置示例：

```json
{
  "mcpServers": {
    "mdut": {
      "type": "http",
      "url": "http://127.0.0.1:5567/mcp",
      "headers": {
        "Authorization": "Bearer 你的强随机Token"
      }
    }
  }
}
```

### 5.2 stdio 模式

```bash
java -jar mdut.jar --mcp
```

无 JavaFX 环境可以使用：

```bash
java -cp mdut.jar mcp.StdioMcpServer
```

stdio 每行处理一条 UTF-8 JSON-RPC 消息，协议响应仅写入标准输出，运行日志写入标准错误；输入关闭后自动释放数据库及本地执行器会话。

客户端配置示例：

```json
{
  "mcpServers": {
    "mdut": {
      "command": "java",
      "args": [
        "-jar",
        "/path/to/MDUT-Extend-v1.4.0-lingxisec/mdut.jar",
        "--mcp"
      ]
    }
  }
}
```

### 5.3 MCP 调用流程

1. 使用 `db_list` 查看已保存连接，或通过 `db_add` 新增连接；
2. 使用 `session_connect` 建立数据库会话；
3. 调用 `db_info`、`db_exec`、文件工具或 `plugin_call`；
4. 使用 `session_disconnect` 释放会话。

| 分类           | 工具                                                    |
| -------------- | ------------------------------------------------------- |
| 连接管理       | `db_list` `db_add` `db_delete` `db_info`                |
| 会话管理       | `session_connect` `session_disconnect` `session_status` |
| SQL            | `db_exec`                                               |
| 文件           | `file_list` `file_read` `file_write` `file_delete`      |
| 数据库专属能力 | `plugin_list` `plugin_call`                             |
| 提权流程       | `priv_escalate`                                         |
| 代理管理       | `proxy_list` `proxy_add` `proxy_delete`                 |

数据库专属能力通过 `plugin_list` 发现，再使用 `plugin_call` 调用。调用前应阅读插件返回的前置条件、适用范围和清理说明。

Redis DoubleFree/TDigest 的 MCP 生命周期：

```text
redis_doublefree_preflight
→ redis_doublefree_start
→ redis_doublefree_command / redis_doublefree_restore_command
→ redis_doublefree_output
→ redis_doublefree_stop
```

断开数据库会话时会自动停止与该会话关联的本地执行器。

## 六、安全建议

- MCP HTTP 默认绑定 `127.0.0.1`，不要直接暴露到公网；确需开放其他网卡时必须使用强随机 Token，并配置额外网络访问控制。
- 不要提交或分享真实 `data.db`、Token、数据库密码和代理认证信息。
- DoubleFree、TDigest、提权、写文件和反弹连接等操作可能改变目标状态，只能在明确授权的范围和维护窗口内使用。
- 发布前建议校验 JAR 和插件来源。社区曾披露部分 MDUT-Extend 发布包的供应链风险，可参考[相关分析](https://www.gm7.org/archives/61364)。

## 七、关于凌曦安全

- 团队官网：[https://www.lingxisec.com/](https://www.lingxisec.com/)
- 微信公众号：凌曦安全
- 联系邮箱：[contact@lingxisec.com](mailto:contact@lingxisec.com)
- 课程介绍：[凌曦安全课程介绍](https://www.yuque.com/syst1m-/blog/lc3k6elv0zqhdal3)

## 八、特别鸣谢

感谢 Calmgh06、Xcaiji6、DoweHFlower、lang、yanxisec 五位师傅在本项目开发与持续完善过程中给予的支持与帮助。

## 九、项目来源

- 上游项目：[DeEpinGh0st/MDUT-Extend-Release](https://github.com/DeEpinGh0st/MDUT-Extend-Release)

## 十、版本说明

### 10.1 校验信息

发布包中的 `mdut.jar` SHA-256：

```text
f10f50695cb29f6fd44921400205eea1ca58653af1faeea4cd69ec03219ee856
```

建议解压后自行核对：

```bash
# macOS / Linux
shasum -a 256 mdut.jar

# Windows
certutil -hashfile mdut.jar SHA256
```

## 十一、安全声明

本项目仅供已获得合法、明确授权的渗透测试、安全研究、防御验证、CTF、靶场、竞赛沙箱、内部测试和安全教育使用。

继续使用前，请确认：

1. 已取得测试目标所有者或合法管理者的明确授权；
2. 所有操作均严格限定在授权范围、期限和规则内；
3. 遵守适用的法律法规及组织安全规范。

严禁将本项目用于任何未授权访问、数据窃取、系统破坏或其他违法违规活动。因超出授权范围或不当使用产生的责任与后果，由使用者自行承担。
