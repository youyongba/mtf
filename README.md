# MTF 多周期共振

保留 W、D、4H、1H、15M、5M、1M 七周期评分、共振、切换交易对及可选飞书通知，不依赖其它系统。

此目录为完整项目，可独立复制到其它服务器。`lib/` 等代码是本项目自己的文件，不依赖上级目录、共享服务或其它四个项目。

## 启动

推荐 Node.js 24 LTS，最低 20。在本目录运行：

```bash
npm ci
cp .env.example .env
npm test
npm start
```

默认地址 `http://127.0.0.1:3102/`；原页面路径 `/mtf.html` 仍可访问。
默认不带凭证、不复制原 `.env`、不自动迁移存档。代理仅通过终端环境变量设置，不写入代码或 `.env`。

## 配置

| 参数 | 默认值 | 说明 |
| --- | --- | --- |
| HOST | 127.0.0.1 | 监听地址，公网使用反向代理 |
| PORT | 3102 | 本项目端口 |
| CONFIG_AUTH_TOKEN | 空 | 管理接口令牌，不能使用公开示例值 |
| BINANCE_TIMEOUT_MS | 10000 | 行情请求超时 |

其它业务变量见 `.env.example` 和各模块源代码，原字段拼写保持不变。通知凭证为空时不发送通知；外部 AI 功能仍需要配置的服务可用。

## 接口

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| GET | /healthz | 进程存活，不代表交易所数据就绪 |
| GET | /api/system | 系统身份及是否启用实盘 |
| GET | /api/mtf/status?symbol=BTCUSDT | 多周期状态 |
| POST | /api/mtf/refresh?symbol=BTCUSDT | 刷新分析 |

原业务接口的请求结构、字段名、鉴权方式保持不变。上游未就绪时返回原有 `ok:false` 或 503；行情错误不伪造为有效数据。

## 数据

所属存档：

- MTF评分为内存快照，启动后重新从交易所采集，不迁移交易存档。


在首次启动前预览迁移，工具不导入 `.env`、不覆盖已有目标文件：

```bash
node scripts/migrate-state.js --from /absolute/old-project
# 仅在源、目标均已停止写入后：
node scripts/migrate-state.js --from /absolute/old-project --apply --source-stopped
```

工具原样复制并校验SHA-256，保留源文件。发现目标已有文件、目标正在运行、非法JSON或路径越界时拒绝执行。旧自定义路径需先整理停机快照到 `system.json` 列出的标准路径。新配置的数据路径不得越出本项目。

## 部署

```bash
pm2 start ecosystem.config.cjs
pm2 restart mtf
```

只允许单个Fork实例，禁止Cluster。PM2的cwd固定为当前项目目录。不要让nodemon监视状态文件，也不要在实盘使用开发热重载。
`nginx.conf.example` 提供本系统专用域名反向代理模板，需配置域名与TLS。服务和代理由操作人员手动启动。

## 验证

`npm test` 在临时目录中运行，不接触本目录或旧系统的真实交易状态。涵盖独立运行、接口、断网降级、路径隔离、进程锁与迁移防覆盖。
生产入口不加载测试网络替身。实盘端到端成交与云主机网络仍需部署后核对，不能用离线测试代替。
