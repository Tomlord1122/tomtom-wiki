---
title: 淺談保險觀念 letf.com.tw 文章分類盤點
source_url: https://letf.com.tw/wp-json/wp/v2/categories?per_page=100&_fields=id,count,name,slug
captured_at: 2026-07-16
---

本次盤點透過網站公開的 WordPress REST API 取得分類資料。主要分類與文章數量如下（分類間可能重疊，不能直接相加）：

- 【槓桿ETF】：172
- 【保險觀念】：117
- 【投資心得】：89
- 【個人成長】：58
- 【理賠爭議】：62
- 【醫療／實支】：55
- 【閱讀筆記】：36
- 槓桿思維：36
- 槓桿系列：27
- 生命週期法：24
- 股息迷思：24
- 汽車保險：23
- 意外保險：21
- 0050 VS 0056：20
- 壽險／儲蓄：19
- 失能／長照：15
- 現金流管理：15
- 再平衡策略：11
- 台股期貨逆價差：11

網站提供 WordPress posts API，可用 `https://letf.com.tw/wp-json/wp/v2/posts` 取得文章 metadata 與完整 HTML 內容。完整 ingest 需要分批進行，並以文章 URL 去重；本盤點不是全部文章的內容摘錄。
