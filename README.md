# Glados自动签到

> 基于 GitHub Actions 的 GLaDOS 自动签到工具，支持多账号与多通道推送（PushDeer / Server酱）。

## 文件结构

```shell
│  checkin.py          # 签到主脚本
│  logging_config.py   # 日志配置（北京时间、[I/W/E] 短标签）
│  requirements.txt    # Python 依赖
│
├─.github
│  └─workflows
│          gladosCheck.yml   # Actions 配置文件
```

## 食用方式

### 注册 GLaDOS 账号

注册地址：<https://glados.space/landing/0A58E-NV28S-6U3QV-33VMG>

- 我的邀请码：[`0A58E-NV28S-6U3QV-33VMG`](https://0a58e-nv28s-6u3qv-33vmg.glados.space)
- 我的优惠码（9 折）：[`DEVILSTORE`](https://0a58e-nv28s-6u3qv-33vmg.glados.space)

### Fork 本仓库

![图片加载失败](imgs/1.png)

### 配置 Secrets

跳转至自己仓库的 `Settings` → `Secrets and variables` → `Action`，按需添加以下 `repository secret`：

#### 3.1 GLADOS_COOKIES（必填）

对应 GLaDOS 账号 Cookie 中的有效部分。**2026-09 起 GLaDOS 已把会话 Cookie 从 `koa:sess` 改为 `gld:sess`，只复制旧版 `koa:sess` 会返回“没有权限”。**

获取方式：

- 重新登录 <https://glados.cloud>（必须重新登录一次，让服务端下发新版 Cookie）
- 按 `F12`，切换到 `Network` 页面下，刷新
- 点击任意请求（如 `status` 或 `checkin`），在 `Request Headers` 下找到 `Cookie`，复制完整值

  > 参考格式：`gld:sess=eyJ1c2xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxAwMH0=; gld:sess.sig=xJkOxxxxxxxxxxxxxxxtnM; koa:sess=...; koa:sess.sig=...;`
  >
  > 重点是必须包含 `gld:sess` 与 `gld:sess.sig`，建议把四个 Cookie 全部带上。

- 多账号请在 `COOKIES` 中添加多个 cookies，中间使用 `&` 连接即可（例如：`c1&c2&c3`）

![图片加载失败](imgs/2.png)
![图片加载失败](imgs/3.png)

#### 3.2 GLADOS_UA（建议填写）

GLaDOS 会校验请求的 User-Agent 是否与登录设备一致，不一致时也会返回“没有权限”。

- 在获取 Cookie 的同一个浏览器控制台执行 `navigator.userAgent`，把输出原样填入 Secret `GLADOS_UA`
- 不填时脚本会依次尝试 Windows / macOS / Linux / iPhone / Android 的标准 UA；如果登录设备是 Firefox 等非 Chrome 浏览器，自动重试可能仍不匹配，建议手动填写
- 浏览器大版本更新后 UA 会变化，如果再次出现“没有权限”，重新登录并同时更新 Cookie 与 `GLADOS_UA`

#### 3.3 GLADOS_DOMAINS（非必填）

默认会依次尝试 `glados.cloud` 和 `railgun.info`。如需只跑某个域名或增加镜像，可配置 Secret `GLADOS_DOMAINS`，多个域名用逗号分隔，例如：

```text
glados.cloud,railgun.info
```

#### 3.4 GLADOS_EXCHANGE_PLAN（非必填）

配置自动兑换积分策略：

| 值 | 积分要求 | 兑换天数 |
|---|---------|---------|
| `plan100` | 100 积分 | 10 天 |
| `plan200` | 200 积分 | 30 天 |
| `plan500` | 500 积分 | 100 天 |

> 不配置时默认不进行自动兑换；如需兑换，设置 `GLADOS_EXCHANGE_PLAN` 为 `plan100` / `plan200` / `plan500`。

#### 3.5 手机推送（非必填）

支持多个推送后端，可在 `Settings`→`Secrets and variables`→`Action` 中按需配置，**任一后端配置后即会推送，支持同时配置多个（都发）**：

- **PushDeer**：添加 `repository secret` 命名为 `PUSHDEER_SENDKEY`，其值对应 PushDeer key（[获取地址](https://www.pushdeer.com/product.html)）。
- **Server酱**：添加 `repository secret` 命名为 `SERVERCHAN_SENDKEY`，其值对应 Server酱 SendKey（[获取地址](https://sct.ftqq.com/)）。

### Star 本仓库

![图片加载失败](imgs/4.png)

## 更新日志

- **2026-01**: 重构代码，添加日志输出方便定位，支持新版网址，支持配置积分兑换策略。
- **2026-04**: 优化代码逻辑与日志输出，支持[新版域名](https://railgun.info)，在 `GLADOS_COOKIES` 中添加新版域名下的 cookies 即可使用。
- **2026-09**: 设备平台自适应重试（无需手动配置 UA 即可自动恢复签到）；网络层自动重试与退避，降低 CI 网络抖动导致的误报失败；日志时间统一为北京时间，定时任务调整为北京时间每天 2 次并避开整点高峰；默认关闭自动兑换，需显式配置后才启用。
- **2026-10**: 日志级别短标签 `[I]/[W]/[E]` ；推送兼容 Server酱（策略模式多后端「都发」），新增 `requirements.txt` 。
- **2026-11**: 适配 GLaDOS 新版 `gld:sess` 会话；支持 `GLADOS_UA` 自定义设备 UA 与 `GLADOS_DOMAINS` 自定义域名；旧版 `koa:sess` Cookie 会给出明确告警；全部签到失败时工作流会标记为失败，不再“假成功”。

## 问题排查与定位

- **日志提示“无权限 (code=-2)” / “没有权限”**：Cookie 不是新版 `gld:sess`，或请求 UA 与登录设备不一致。请重新登录 <https://glados.cloud>，按 3.1 重新复制完整 Cookie，并按 3.2 配置 `GLADOS_UA` 后重新运行。
- **工作流显示红色失败**：说明本轮所有账号/域名都签到失败，请按上面的日志提示处理；只要有一个账号成功或重复签到，工作流仍为绿色。
- 其他问题可通过查询 Actions 中的 running checkin 日志快速定位。

  <img width="1684" height="844" alt="image" src="https://github.com/user-attachments/assets/45348a5f-43e4-45f5-8fdf-ce84d343b30d" />

## 声明

本项目不保证稳定运行与更新，因 GitHub 相关规定可能会删库，请注意备份。
