# 中转站调度原型

Vue 3 + Vite 搭建的中转站运营调度原型，包含站点负载、波次进度、站内流向、异常决策和发车线路看板。

## 本地运行

```bash
npm install
npm run dev
```

## 构建

```bash
npm run build
```

构建输出目录为 `dist`。

## Cloudflare Pages

仓库已包含 GitHub Actions 工作流：`.github/workflows/deploy-cloudflare-pages.yml`。

需要在 GitHub 仓库 Settings -> Secrets and variables -> Actions 中配置：

- `CLOUDFLARE_API_TOKEN`
- `CLOUDFLARE_ACCOUNT_ID`

Cloudflare Pages 项目名使用 `zhongzhuan-prototype`，发布目录为 `dist`。

工作流会在推送到 `main` 时构建并部署；如果 Pages 项目尚不存在，会先尝试创建 Direct Upload 项目。
