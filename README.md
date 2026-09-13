# smgtv-live

看看新闻（SMG / live.kankanews.com）网页直播增强油猴脚本。在既有脚本基础上，改为**被动截获站点自身响应中的加密播放地址并解密注入**，规避被服务端拦截的自取流请求，并在页面右上角显示取源状态。

> 本仓库内容仅供学习交流。

## 安装

1. 浏览器安装 [Tampermonkey](https://www.tampermonkey.net/)（推荐）
2. 点此安装脚本：[**smg_fivestar.user.js**](https://raw.githubusercontent.com/shangphoenix/smgtv-live/main/smg_fivestar.user.js)
3. 打开 [SMG 直播页](https://live.kankanews.com/huikan?id=10)，选择频道观看

脚本已内置 `@updateURL` / `@downloadURL`，Tampermonkey 会从本仓库自动检查更新。

## 工作原理

看看新闻的播放地址（`channel_info.live_address` / `shift_address`）由内部接口 `kapi.kankanews.com/content/pc/tv/*` 下发，字段为 RSA 加密串，用固定公钥经 `JSEncrypt` 公钥解密后得到火山源 m3u8（`volc-stream.kksmg.com/live/<code>/index.m3u8?token=<JWT>`）。

- **旧脚本**：自行发起签名接口请求（donor 扫描）去取地址。该请求现常被服务端拦截（`Failed to fetch`），导致取不到源。
- **本脚本**：优先**被动读取站点自身**已成功返回的响应，解密其中地址并注入播放器；自取流仅作为兜底。
- 保留上游 PR 的 CDN 签名到期续期（JWT `exp` 与 `volcTime` 取较早者，到期前约 2 分钟续期）、回放进度拖动、重复初始化保护与网络错误重试。

## 状态提示条（右上角）

| 颜色 | 含义 |
|------|------|
| ⏳ 灰 | 正在获取直播源 |
| ✅ 绿 | 已取到直播源（数秒后自动淡出） |
| ⚠️ 黄 | 接口被限流 / 暂无可用源，请稍后重试或换个时段 |

## 已知限制

播放地址完全依赖服务端下发。当**服务端整体拒发地址**（连网站自身请求都 `Failed to fetch`、页面显示"暂无节目单"）时，任何纯前端脚本都无源可用——此为服务端封锁，需等待恢复或更换时段。

## 致谢

基于以下前人工作：
- [jolin1314joker/SMG_TV](https://github.com/jolin1314joker/SMG_TV)
- krfalcon（roies）在 SMG_TV 的 CDN 续期修复 PR
- [Popukok/smg_live](https://github.com/Popukok/smg_live)

## License

[MIT](./LICENSE)
