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
