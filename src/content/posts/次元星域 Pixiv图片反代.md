---
published: 2026-10-20
title: 次元星域 Pixiv图片反代
pinned: false
description: 什么叫之前的 Pixiv反代代码出现525错误？看我绘里酱编写出一个新代码，按最新的方案进行反代！
tags: [Markdown, Pixiv]
category: 教程
image: https://img.107211.xyz/file/img/9KHOc1zw.webp
---
## 前言
什么叫之前的 Pixiv反代代码出现525错误？看我绘里酱编写出一个新代码，按最新的方案进行反代！

## 示例站点
[次元星域 Pixiv图片反代站](https://pximg.107211.xyz)

### 部署方法
注册登录 Cloudflare，创建一个worker，配置自定义域名，将以下代码输入并部署，访问域名即可使用

使用格式参考示例图片

### 代码

```bash
/**
 * Pixiv 图片反代 - Cloudflare Workers 单文件版
 * 效果: https://<域名>/img-master/img/2026/10/17/00/00/18/150930518_p0_master1200.jpg
 * 直接粘进 Workers 编辑器并 Deploy。
 */
const UPSTREAMS = ["https://i.pixiv.re", "https://i.pximg.net"];
const CORS = {
  "access-control-allow-origin": "*",
  "access-control-allow-methods": "GET,HEAD,OPTIONS",
  "access-control-allow-headers": "*",
};
const PAGE = `<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Pixiv 图片反代</title>
<link rel="stylesheet" href="https://fonts.loli.net/css2?family=JetBrains+Mono:wght@400;700&display=swap">
<style>
	*{margin:0;padding:0;box-sizing:border-box}
	body{min-height:100vh;font-family:"JetBrains Mono","Maple Mono",-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Arial,sans-serif;background:#fff url(https://t.alcy.cc/ycy) center/cover fixed no-repeat}
	@media(orientation:portrait){body{background-image:url(https://t.alcy.cc/mp)}}
	#blur-overlay{position:fixed;inset:0;background:linear-gradient(135deg,rgba(255,183,197,.08) 0,rgba(255,143,171,.12) 100%);backdrop-filter:blur(3px);z-index:-1;pointer-events:none}
	.page-wrapper{position:fixed;inset:0;display:flex;justify-content:center;align-items:center;padding:20px;overflow-y:auto;z-index:1}
	.card-container{position:relative;overflow:hidden;width:100%;max-width:440px;padding:50px 40px;text-align:center;border-radius:24px;background:rgba(255,255,255,.9);backdrop-filter:blur(20px);box-shadow:0 20px 60px rgba(0,0,0,.3),0 0 0 1px rgba(255,255,255,.5) inset;animation:containerAppear 1s ease-out .3s backwards}
	.card-container::before{content:'';position:absolute;top:0;left:0;right:0;height:4px;background:linear-gradient(90deg,#ffb7c5 0,#fff 50%,#ffb7c5 100%);background-size:200% 100%;animation:gradientShift 3s ease infinite}
	@keyframes containerAppear{0%{opacity:0;transform:translateY(30px) scale(.9)}100%{opacity:1;transform:translateY(0) scale(1)}}
	@keyframes gradientShift{0%{background-position:0 50%}50%{background-position:100% 50%}100%{background-position:0 50%}}
	.page-icon{margin:0 auto 20px;animation:fadeIn .8s ease-out .8s backwards}
	.page-icon img{display:block;width:100%;border-radius:16px}
	@keyframes fadeIn{from{opacity:0}to{opacity:1}}
	.page-title{font-size:28px;font-weight:700;color:#ffb7c5;line-height:1.2;letter-spacing:-.5px}
	.error-sub{display:block;margin-top:24px;padding:20px 24px;border-radius:12px;background:rgba(255,183,197,.18);color:#666;font-size:14px;font-weight:500;line-height:1.5;word-break:break-all;box-shadow:0 4px 15px rgba(0,0,0,.08);animation:fadeIn .3s ease-out}
	.error-sub a{color:#ff8fab}
	.footer-hint{margin-top:50px;font-size:13px;color:#999;text-align:center;animation:fadeIn 1s ease-out 1s backwards}
	.footer-hint a{color:#ffb7c5;text-decoration:none;font-weight:500}
	@media(max-width:480px){.card-container{padding:35px 24px;border-radius:18px}.page-title{font-size:24px}}
</style>
</head>
<body>
<div id="blur-overlay"></div>
<div class="page-wrapper">
	<div class="card-container">
		<div class="page-icon"><img src="https://img.107211.xyz/file/img/yP9XFpEh.webp" alt="Pixiv"></div>
		<h1 class="page-title">Pixiv图片反代</h1>
		<span class="error-sub">用法如:<br><a href="https://pximg.107211.xyz/img-master/img/2026/10/17/00/00/18/150930518_p0_master1200.jpg">https://pximg.107211.xyz/img-master/img/2026/10/17/00/00/18/150930518_p0_master1200.jpg</a></span>
		<div class="footer-hint">由 <a href="https://pximg.107211.xyz">Pixiv图片反代</a> 强力驱动</div>
	</div>
</div>
</body>
</html>`;

export default {
  async fetch(req) {
    const { pathname, search } = new URL(req.url);
    if (req.method === "OPTIONS") return new Response(null, { headers: CORS });
    if (pathname === "/")
      return new Response(PAGE, { headers: { "content-type": "text/html; charset=utf-8" } });

    const init = {
      headers: {
        Referer: "https://www.pixiv.net/",
        "User-Agent":
          "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0 Safari/537.36",
        Accept: "image/avif,image/webp,image/*,*/*;q=0.8",
      },
      cf: { cacheTtl: 86400, cacheEverything: true },
    };

    for (const base of UPSTREAMS) {
      try {
        const r = await fetch(base + pathname + search, init);
        if (r.status >= 500) continue;
        const headers = new Headers(r.headers);
        headers.set("access-control-allow-origin", "*");
        if (!headers.has("cache-control")) headers.set("cache-control", "public, max-age=86400");
        return new Response(r.body, { status: r.status, headers });
      } catch {}
    }
    return new Response("upstream error", { status: 502, headers: CORS });
  },
};
```

### 协议&许可证
MIT

## 联系及反馈
有问题可在底下评论区或联系邮箱[ciallo@107211.xyz](mailto:ciallo@107211.xyz)