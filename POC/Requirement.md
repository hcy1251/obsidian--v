## 🔹 **Obsidian Plugin UX 設計**

### **📌 1. 插件啟動方式**

✅ **Sidebar Plugin Icon**

- 在 **左側 Sidebar 加入 Plugin Icon**，點擊後開啟插件操作界面。

✅ **命令面板（Command Palette）支援**

- 可透過 `Ctrl/Cmd + P` 搜尋插件名稱來觸發。

✅ **快捷鍵綁定（可選）**

- 允許用戶自訂快捷鍵（如 `Ctrl + Alt + G`）來直接執行 Plugin。

---

### **📌 2. Plugin UI 操作流程**

**🔸 插件介面：Dropdown + 生成按鈕**  
點擊 Plugin Icon，會彈出一個側邊視窗，包含：

- **Dropdown 選擇 `task`**（提供不同 LLM 任務選項）
- **「生成內容」按鈕**（點擊後觸發 API）
- **可選：在 UI 上顯示原始內容摘要**（幫助用戶理解當前處理的文件）

```
-----------------------------------
|  🧠 AI 內容生成插件             |
-----------------------------------
| 🔽 Task 選擇:                   |
| [Summarize Content]  ⌄         |
|                                  |
| 📝 當前文件: filename.md         |
|                                  |
| 🔄 [ 生成內容 ] (Button)         |
-----------------------------------
```

---

### **📌 3. 內容生成與文件處理**

1️⃣ **當用戶選擇 `task` 並點擊「生成內容」**

- 讀取當前 Markdown 文件內容
- 根據 `task` 與 `origin_content` 呼叫 API
- API **逐字輸出**，即時寫入 `{origin_file_name}_ai.md`
- 插件 UI 顯示 **進度條**，讓用戶知道處理進行中

2️⃣ **生成完成後，右側開啟新文件 `{origin_file_name}_ai.md`**

- 插件 UI 彈出 **「後續操作」選擇框**
- 選擇：
    - ✅ **Keep**（保留新文件）
    - 🔄 **Replace**（覆蓋原始文件）
    - 🗑 **Delete**（刪除新文件）

```
------------------------------------------------
| ✅ 內容已生成，請選擇後續操作：              |
|----------------------------------------------|
| 🔄 Replace  |  🗑 Delete  |  ✅ Keep        |
------------------------------------------------
```

---

## 🔹 **UX 設計考量**

1. **即時輸出 & 進度條**
    
    - 因為 API 是逐字輸出，UI 需要一個「Loading」效果，防止使用者以為卡住。
    - 建議在 `{origin_file_name}_ai.md` 寫入時，顯示 **「內容生成中...」**
2. **Task 設定提供更多彈性**
    
    - `task` 選單應該可以透過設定擴充，例如 `tasks.json` 定義可選值：
        
        ```json
        {
          "tasks": [
            { "id": "summarize", "label": "Summarize Content" },
            { "id": "rewrite", "label": "Rewrite in Formal Style" },
            { "id": "expand", "label": "Expand with Examples" }
          ]
        }
        ```
        
    - 這樣能允許未來新增更多 `task`，而不必改動核心程式。
3. **Plugin 操作需簡潔流暢**
    
    - 避免多餘的步驟，例如每次都彈出確認視窗會影響效率，應該讓用戶一次完成選擇。
    - 如果 `task` 選擇是 **固定常用的幾個**，可以考慮：
        - **最近使用**（記住上次選擇的 `task`）
        - **快速鍵操作**（如 `Alt + 1` = Summarize，`Alt + 2` = Rewrite）

---

## 🔹 **技術實現**

4. **Obsidian 插件 UI**
    
    - 使用 `Modal` 或 `View` 來實現選擇 `task` 的 UI
    - 使用 `Notice` 來顯示進度與結果
    - 使用 `DropdownComponent` 來選擇 `task`
5. **API 請求**
    
    - 透過 `fetch` 呼叫 API，並逐字寫入 `{origin_file_name}_ai.md`
    - 確保非同步請求不會影響 Obsidian 其他操作
6. **文件處理**
    
    - 透過 `app.vault` 來創建與修改 `.md` 文件
    - 透過 `app.workspace.activeLeaf` 打開新文件
    - `app.workspace.openLinkText(filename, '', true)` 來確保文件正確打開

