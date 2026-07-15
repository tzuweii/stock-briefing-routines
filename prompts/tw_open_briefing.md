每個台灣平日09:30執行:確認今天是否為台股交易日(排除週末、國定假日、颱風假),休市則結束任務不產出內容。

若為交易日,蒐集以下資料並產出報告:

【報價】
用Bash執行帶標準瀏覽器User-Agent的HTTP請求讀取以下端點(加range=1d&interval=1d參數,只取當天單一快照):
https://query1.finance.yahoo.com/v8/finance/chart/0052.TW?range=1d&interval=1d
注意:內建WebFetch工具會被Yahoo反爬蟲機制擋回403,需改用Bash/curl並帶瀏覽器UA(例如"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0 Safari/537.36")才能正常取得資料。若query1.finance.yahoo.com失敗,可改試query2.finance.yahoo.com同路徑。
只讀取回傳JSON裡chart.result[0].meta物件的regularMarketPrice與chartPreviousClose(注意:欄位名稱是chartPreviousClose,不是previousClose,meta物件裡沒有後者),計算漲跌%。

【新聞來源設定】
1. 中央社財經 RSS(優先):https://feeds.feedburner.com/rsscna/finance
2. 經濟日報 RSS(國際財經分類):https://money.udn.com/rssfeed/news/1001/5588
3. Yahoo股市台股動態 RSS(補充):https://tw.stock.yahoo.com/rss?category=tw-market
用WebFetch讀取以上三個端點,解析<item>列表(title、link、pubDate、description)。

【時間窗口 — 鎖定「開盤前」】
只保留pubDate落在「當日早上05:00」到「查詢當下」這段區間的新聞,窗口外的新聞一律捨棄。

【新聞】
用web_search補充,不超過5次查詢,聚焦:
- 亞洲盤早盤動向(日經、韓股,尤其半導體同業)
- 影響今日開盤情緒的總經事件(Fed、地緣政治、匯率)
- 與科技股高度相關的消息
同樣只採用時間窗口內發布的內容。

【Top Call — 今日開盤前三大關注事件】
從候選事件中選出三則對「今天開盤走勢」最具參考價值的事件,依重要性排序並標明序號(1/2/3),每則一句話。每則需附至少兩個獨立來源佐證(同一則通稿被轉載不算兩個來源);找不到第二來源可列入但標註「單一來源,待進一步查證」。內容應聚焦「將如何影響今天開盤」,而非回顧昨天已經發生、市場已消化的事。

【HTML排版 — 使用固定樣板】
先用notion-fetch讀取樣板頁面(page_id: 39c99975-636b-8179-816e-cfe1784dd04a),取得其中code block裡的完整HTML原始碼。
將以下{{佔位符}}替換成當天實際內容,不得修改CSS、class名稱、HTML結構或任何樣式屬性:
- {{PAGE_TITLE}}: 台股開盤briefing [今天日期]
- {{DATE}}: 今天日期 / {{REPORT_ICON}}: 📊 / {{REPORT_TITLE}}: 台股開盤briefing
- {{COLOR_UP}}: #d0342c / {{COLOR_DOWN}}: #1a7d3c(台股慣例,漲紅跌綠)
- {{TOP_CALL_HEADING}}: 今日開盤前三大關注事件 / {{TOP_CALL_ITEMS}}: 三則事件,含序號、一句話、來源標註
- {{INDEX_CARDS}}: 只需1張加權指數卡片
- {{TIME_TAG}}: 盤中即時(09:30)
- {{DYNAMICS_HEADING}}: 市場動態 / {{DYNAMICS_ITEMS}}: 3-5點,涵蓋大盤走勢、總經消息、產業焦點
- {{WATCH_HEADING}}: 本週關注 / {{WATCH_ITEMS}}: 2-4點,只列已知事件,不預測方向
- {{HOLDING_NAME}}: 0052富邦科技 / {{HOLDING_SYMBOL}}: 0052.TW / {{HOLDING_PRICE}} {{HOLDING_CHANGE}}: 現價與漲跌 / {{HOLDING_NOTES}}: 1-3點相關消息;無消息寫「跟隨大盤與半導體產業走勢,無ETF專屬事件」
- {{UP_OR_DOWN}}: 依實際漲跌對應class(up/down)
- {{RISK_ITEMS}}: 2-4項風險因素
- {{TRADE_IDEA_TEXT}}: 一行加碼觀察點(存股邏輯,非賣出建議);無訊號寫「維持觀察」

上傳置換後的完整HTML,不再自行設計CSS或排版結構。

【執行摘要 — 僅異常時填寫】
若執行過程中出現工具失敗、fallback、資料缺口或需自行判斷取捨的情況,依序記錄「步驟/發生什麼/如何處理」寫入Notion database的「執行摘要」欄位。全程順利則留空,不寫"一切正常"。

用詞紀律:只寫可查證事實;區分「傳言/公司說法」與「已確認事實」;因果用「主要受到...影響」,不用「一定會/即將大漲」等推測語;證據不足就寫「目前無法確認」,不硬湊內容。

寫入Notion database(202fe0da-d4a3-44ee-a96c-d64bf12eec93):
- Name:台股開盤briefing [今天日期]
- 日期:今天日期 / 市場:台股 / 報告類型:開盤前情報
- 內容:用create-attachment上傳置換後的HTML,embed區塊嵌入
- 執行摘要:若有記錄則填入,無異常則留空不填

寫入失敗則輸出完整HTML並註明「寫入失敗,請人工確認」。用繁體中文。
