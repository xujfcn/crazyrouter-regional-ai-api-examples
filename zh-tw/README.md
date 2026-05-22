# 一個 API Key 呼叫 GPT、Claude、Gemini：5 分鐘設定教學

如果你正在台灣開發 AI 產品，常見情境可能是：一開始先接 OpenAI，後來想測 Claude 的長文理解，再測 Gemini 的速度或多模態能力。問題是，每多接一個供應商，就多一組 API key、一套 SDK、一種錯誤格式與一份維護成本。

比較實際的做法，是把模型供應商整合到同一個 OpenAI 相容 API gateway 後面。你的應用程式只需要記住一個 endpoint 和一個 API key，再透過 `model` 參數切換 GPT、Claude 或 Gemini。

這篇教學會用 Crazyrouter 示範一個簡單、可落地的設定方式。

## 適合什麼情境？

這種做法特別適合：

- 想在同一個產品流程中比較不同模型。
- 已經使用 OpenAI SDK，但希望測試 Claude 或 Gemini。
- 不想在後端管理多組供應商密鑰。
- 希望之後加入 fallback model，提升穩定性。
- 團隊想先快速驗證，再決定長期模型策略。

## 1. 建立 API Key

先閱讀文件介紹，再照 quickstart 建立 API key：

- 文件介紹：[Crazyrouter docs](https://docs.crazyrouter.com/introduction?utm_source=taiwan&utm_medium=regional_auto&utm_campaign=docs_intro)
- 快速開始：[Crazyrouter quickstart](https://docs.crazyrouter.com/quickstart?utm_source=taiwan&utm_medium=regional_auto&utm_campaign=quickstart)

在本機開發時，可以先放到環境變數：

```bash
export CRAZYROUTER_API_KEY="cr_..."
```

正式環境請放在部署平台的 secret manager，不要寫死在程式碼，也不要放到前端。

## 2. 設定 OpenAI 相容 endpoint

如果你使用 Node.js，先安裝 OpenAI SDK：

```bash
npm install openai
```

接著建立 client，重點是設定 `baseURL`：

```js
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.CRAZYROUTER_API_KEY,
  baseURL: "https://crazyrouter.com/v1",
});

const completion = await client.chat.completions.create({
  model: "openai/gpt-4o-mini",
  messages: [
    { role: "system", content: "你是簡潔清楚的技術助理。" },
    { role: "user", content: "用三句話解釋 AI API gateway 的用途。" },
  ],
});

console.log(completion.choices[0].message.content);
```

如果你原本的程式已經使用 OpenAI SDK，通常只需要改 API key 與 `baseURL`，不一定要大改架構。

## 3. 切換 GPT、Claude、Gemini

接著可以用同一段 prompt 測不同模型：

```js
const messages = [
  { role: "user", content: "幫我把這段客服訊息整理成一句摘要：客戶反映付款頁面一直逾時。" },
];

const models = [
  "openai/gpt-4o-mini",
  "anthropic/claude-3-5-haiku",
  "google/gemini-1.5-flash",
];

for (const model of models) {
  const result = await client.chat.completions.create({ model, messages });
  console.log(`\n--- ${model} ---`);
  console.log(result.choices[0].message.content);
}
```

這樣做的好處是，你可以用真實產品資料評估模型，而不是只看 benchmark。

## 4. Python 範例

Python 的設定也很接近：

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["CRAZYROUTER_API_KEY"],
    base_url="https://crazyrouter.com/v1",
)

response = client.chat.completions.create(
    model="anthropic/claude-3-5-haiku",
    messages=[
        {"role": "system", "content": "你是技術文件寫作助理。"},
        {"role": "user", "content": "幫我寫一段 API 更新說明。"},
    ],
)

print(response.choices[0].message.content)
```

## 5. 上線前建議加上的保護

正式接到產品流程前，建議至少加上這幾件事：

1. **Timeout**：避免請求卡住太久。
2. **Retry with backoff**：暫時性錯誤可以重試，但不要無限重試。
3. **Model logging**：記錄模型名稱、延遲、錯誤與大致成本。
4. **Fallback**：非關鍵流程可以準備備援模型。

例如：

```js
async function askWithFallback(messages) {
  const candidates = ["openai/gpt-4o-mini", "google/gemini-1.5-flash"];

  for (const model of candidates) {
    try {
      return await client.chat.completions.create({ model, messages });
    } catch (error) {
      console.warn(`模型失敗：${model}`, error.message);
    }
  }

  throw new Error("所有模型都呼叫失敗");
}
```

## 下一步

先挑一個低風險功能測試，例如摘要、分類、客服草稿、內部報表或搜尋 query rewrite。用相同 prompt 跑 GPT、Claude、Gemini，觀察品質、延遲與成本，再決定預設模型。

完整設定流程可以從 [Crazyrouter quickstart](https://docs.crazyrouter.com/quickstart?utm_source=taiwan&utm_medium=regional_auto&utm_campaign=next_step) 開始。
