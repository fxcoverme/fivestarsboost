# FiveStarsBoost 网站迁移简报 — 从 muse.ai 搬到 fivestarsboost.com 自有托管

准备时间：2026-11-01 晚（ET）。状态：**包已备好、尚未切换 DNS**——切换需 Bernice 本人登录 Porkbun 的那一步，晚上审核时一起做。

## 选定托管：GitHub Pages（免费）

- 理由：Bernice 已有 github.com 账号；免费、自带 HTTPS 证书、稳定；站点是纯静态单页，正合适。
- 备选（都没建号，需要时再说）：
  - **Netlify**：需用邮箱或 GitHub 登录新建账号；DNS 改法简单（apex 用 Netlify 的 A 记录 75.2.60.5 + www CNAME 到站点名）。未经同意不建。
  - **Vercel**：同样需新建账号（GitHub 登录最省事）；apex A 记录 76.76.21.21 + www CNAME cname.vercel-dns.com。未经同意不建。

## 导出完整性核验（2026-11-01）

- `artifact.export` 导出成功：单文件 HTML，2 个资源全部内嵌，0 个未解析资源。
- 核验通过：页头/页脚 logo（5 张 PNG 全部以 data URI 内嵌）、评论回复生成器逻辑（语气 Grateful/Professional/Warm + localStorage 计数）、Google 表格注册链接（docs.google.com/forms/…1FAIpQLScDBxg…/viewform，与现站一致）、三语版本（EN 纯英文 / ES+EN / ZH+EN，含「回评」）、四张价格卡（$0 / $7.99 / $19.99 / $29.99）、页脚 VelaGrid 署名。
- **全文 0 处 muse.ai 引用**，唯一的外部链接是 Google 表格和 Gumroad 产品页（jrbklu）——都是本站本就该有的出站链接。在任何静态托管上都能独立运行。
- 文件大小 2.1 MB（主要是内嵌 logo 图），加载稍慢于现站但可接受。

## 已备好的部署包

目录：`~/workspace/autonomous-income/website-hosting/`（已 git init 并提交，commit 9d4a099）

- `index.html` — 导出成品，自包含，不依赖任何外部文件
- `CNAME` — 内容 `fivestarsboost.com`（GitHub Pages 自定义域名靠它）
- `logo-a.png` / `logo-c-wide.png` — 品牌图（index.html 未引用它们，是内嵌的；留作备用素材）
- `MIGRATION-BRIEF.md` — 本文件

## 还差的一步：GitHub 仓库（需要 Bernice 的 GitHub 登录，一次性，约 2 分钟）

本机（VM）实测：`gh` 未登录、SSH 到 github.com 被网络挡住、无任何已存的 GitHub 凭据——所以仓库还没法从这边建。两个办法任选：

- **办法 A（推荐）**：在 VM 上跑一次 `gh auth login` 设备码流程，Bernice 在浏览器里输一次码授权。之后 MM 一条命令完成：建仓 → 推送 → 开 Pages。
- **办法 B**：用浏览器任务（Bernice 已存的 github.com 登录）建一个空的公开仓库（如 `fivestarsboost-site`），再把仓库地址给 MM；推送仍需要办法 A 的授权，所以最终还是要做一次 A。

仓库建好后的命令（MM 执行，无需再动域名）：
1. `git remote add origin … && git push`
2. 在仓库 Settings → Pages（或 API）启用 Pages：分支 `main`、根目录 `/`；自定义域名读 CNAME 文件自动填 `fivestarsboost.com`
3. 此时站点会先出现在 `<用户名>.github.io/<仓库名>` 或自定义域名待生效状态——**不影响现网**，因为域名 DNS 还指着转发。

## 最后一步：Porkbun DNS 切换（需 Bernice 本人登录 Porkbun 操作，MM 在旁指挥）

现状态：Porkbun 上有一条 **URL Forwarding（301 跳转，含通配）** 把 fivestarsboost.com 和 www 转到 muse.ai 链接。

切换时按顺序做：

1. **删掉 URL Forwarding**（fivestarsboost.com 的转发规则；Porkbun 里开着转发时加不了 apex 的 A 记录，必须先删）。
2. **给根域名（@）加 4 条 A 记录**（GitHub Pages 固定 IP）：
   - `185.199.108.153`
   - `185.199.109.153`
   - `185.199.110.153`
   - `185.199.111.153`
   - （可选 AAAA：`2606:50c0:8000::153`、`2606:50c0:8001::153`、`2606:50c0:8002::153`、`2606:50c0:8003::153`）
3. **给 www 加一条 CNAME 记录**：`www` → `<Bernice 的 GitHub 用户名>.github.io`（切换时从她 GitHub 主页确认确切用户名，别猜）。
4. 等 DNS 生效（通常几分钟到 2 小时，最长 24 小时）后，GitHub 会自动签发 HTTPS 证书；证书就绪后在仓库 Pages 设置里勾上 **Enforce HTTPS**。
5. 验证：直接打开 https://fivestarsboost.com ——地址栏应是自家域名、无 muse 字样、工具可生成回复、注册按钮跳 Google 表格。

注意：**不要动** Porkbun 里的任何邮箱转发（quoteinfo@、admin@ 的 EMAIL 转发是另一套，跟这次无关）。

## 风险

- **传播期短暂不一致**：切换后几分钟到几小时内，不同地区访客可能分别看到新站或被旧缓存带着走；转发删掉的那一刻起，没缓存到新 DNS 的访客会短时打不开——选晚上低峰时段切，影响最小。
- **HTTPS 证书空窗**：证书签发前 http 可开、https 可能报错，一般 1 小时内好；这段时间别对外发链接。
- GitHub Pages 免费版要求仓库公开（站点内容本身就是公开的，无隐私问题）。

## 回滚（随时可退，2 分钟）

1. 在 Porkbun 删掉上面加的 A / CNAME 记录；
2. 重新加回 **URL Forwarding：fivestarsboost.com → https://muse.ai/s/fivestarsboost-wxj6zsxxaxaxoxro/lbOgiDl1wB85H3x_EQ4vpTdchG2ez__ZKiGVhdDjTNs，Permanent (301)，通配开**；
3. muse.ai 那份站点一直没动，等 DNS 缓存过期（≤24 小时，通常快得多）就回到现在的状态。
