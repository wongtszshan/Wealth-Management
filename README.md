# 澄理 · Wealth Management

个人双币种理财 PWA（SGD / CNY）。前端静态托管，数据默认本地存储，支持上传到独立私库的 AES-256-GCM 加密备份。

## 已实现
- 手动记账、修改/删除、账户昵称管理
- CSV 文件多文件批量导入、待确认复核与基础去重
- OCBC 邮件通知正文粘贴解析（支持示例格式；非邮箱自动读取）
- 消费分类可视化、按月/同期对比、分类预算和储蓄目标
- 本地 JSON / CSV 导出、恢复；GitHub Private Repo 客户端加密备份（需用户提供短期使用的 Fine-grained Token）
- 手机端响应式界面与 PWA 缓存

## 尚未实现
- 截图自动 OCR / AI 识别（目前仅可批量选择并预览，不能从截图生成交易）
- 银行 API、邮件自动授权同步、跨设备冲突合并、正式账单自动对账

## 安全
- 前端不包含服务端密钥；GitHub Token 不写入持久存储。
- 私库同步必须由用户输入独立加密短语（12 位以上），在浏览器完成加密后再上传密文。
- 本地账目使用 localStorage，**未加密**。请勿在共用设备上输入敏感交易。
- 不要把真实账单、身份证、令牌提交到公开仓库。
- 私库数据仓库：`wongtszshan/Wealth-Management-data`，备份文件：`finance.encrypted.json`。

## 启用 GitHub Pages
在公库 Settings → Pages → Deploy from a branch → main / (root) 保存。

## CSV 示例
```csv
date,description,amount,type,currency
2026-11-19,7-ELEVEN,3.20,expense,SGD
2026-11-19,午餐,35,expense,CNY
```


## Cloudflare Worker 同步（v1.1）
已在 Cloudflare 创建并部署 `wealth-management-api`，API 地址为：
`https://wealth-management-api.wangtszshan.workers.dev`

Worker 采用应用密码登录（会话默认有效期 30 天），云端 AES-256-GCM 加密后写入私人 GitHub 仓库。网页缓存登录会话，不需要在浏览器输入 GitHub Token。**本地 localStorage 仍未加密。** 云端加密密钥仅保留在 Worker Secret；务必做好该密钥的安全备份和灾难恢复计划。

需要在 Cloudflare Dashboard → Workers & Pages → wealth-management-api → Settings → Variables and Secrets 中增加以下两个 Secret：
- `GITHUB_TOKEN`：Fine-grained PAT，限制到私库 `wongtszshan/Wealth-Management-data`，Contents: Read and Write。
- `APP_PASSWORD`：用于网页 App 首次登录的强独立密码。

另两个 Secret `SESSION_SECRET` 和 `DATA_ENCRYPTION_KEY` 已自动安全生成，**不要随意重置 DATA_ENCRYPTION_KEY，否则将无法解密已有数据**。

在这两个必需 Secret 配置齐全前，Worker 会返回 HTTP 503，云端功能不可用，本地记账不受影响。

同步防护：使用 GitHub content SHA 检查版本，冲突时停止覆盖；新设备本地已有记录且云端不同也会先提示。当前仍不具备自动数据合并功能。登录会话保存在浏览器 localStorage，建议仅在本人可信设备上使用。

警告：请勿在 GitHub 公开仓库、聊天对话或截图中公布密钥 / Token。后续应增加密码尝试频率限制、会话撤销与设备管理。

## AI 图片账单识别（v1.2.0）
网页的「批量导入」页面支持批量选择截图（单次最多 5 张），选择 Kimi K2.5 或 Claude Sonnet 5.5，点击「开始识别」后发送图片。结果进入「待确认交易」，必须人工复核才会写入账本。

需在 `wealth-management-api` → Settings → Variables and Secrets 添加 **Secret**：
- `KIMI_API_KEY`：Moonshot AI 官方 API Key（非其他中转商 Key）
- `ANTHROPIC_API_KEY`：Anthropic 官方 API Key（非其他中转商 Key）

两者按需配置一个或两个。用户已有的其他 Worker 的 Secret 不会自动共享。Worker 的模型调用处使用官方 API 入口和模型 ID，使用量按照各提供商计费。接口要求用户已登录，单次限制 1–5 张图及请求大小；未完成实际模型凭据测试前不能保证真实识别成功。

隐私：截图仅在用户点击「开始识别」后传往 Cloudflare Worker，再发送给所选模型商；模型商可能按照其数据处理政策处理图像。不要上传完整卡号或身份证件照片。原始截图不写入 GitHub 私库。建议进一步实现服务端调用频率与费用上限。

## PDF 银行账单（v1.3.0）
「批量导入」增加原始 PDF 上传入口（最大 20MB），通过浏览器端 pdf.js 转换用户指定的连续 1–5 页为图片，再经已配置的 Kimi / Claude 视觉 API 提取交易。完整 PDF 不写入云端仓库，只有在用户触发识别时才发送所选页面图片。交易先进入待确认列表，不自动记账。密码保护的 PDF 需先在本机解锁。PDF.js 使用 jsDelivr CDN，第一次打开时需要联网；暂未针对具体银行 PDF 验证交易准确率或端到端测试。
