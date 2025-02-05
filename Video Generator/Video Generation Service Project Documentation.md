## 專案概述 (Project Overview)

**Video Generation Service** 是一個可擴充且高效的系統，用於根據提供的字幕與時間資訊，自動產生包含字幕的影片。  
此服務支援多任務並行處理，能夠同時進行多個影片段落的合成，以有效縮短整體生成時間並提升效能。

---

## 專案結構 (Project Structure)

```
video_service/
├── app/
│   ├── core/
│   │   ├── video_processor.py        # 影片合成邏輯
│   │   ├── audio_processor.py        # 音訊生成與處理
│   │   ├── content_processor.py      # 判斷背景素材(圖片/影片)
│   │   └── task_status.py            # 追蹤任務狀態
│   ├── tasks/
│   │   ├── celery_app.py             # Celery 初始化與設定
│   │   ├── video_tasks.py            # 定義 Celery 任務
│   │   └── orchestrator.py           # 工作流程協調 (子任務串接)
│   ├── api/
│   │   └── routes.py                 # FastAPI 路由定義
│   └── config/
│       └── settings.py               # 系統與配置參數
├── data/
│   ├── videos/                       # 原影片素材
│   ├── images/                       # 圖片素材
│   ├── fonts/                        # 字型檔
│   ├── subtitles/                    # 原始字幕
│   ├── processed_subtitles/          # 供處理的字幕檔
│   ├── output/                       # 最終產出影片路徑
│   ├── temp_output/                  # 暫存檔案 (音訊/影片片段等)
│   └── temp/                         # 其他暫時檔案
└── docker/
    ├── Dockerfile
    ├── docker-compose.yml
    └── celery.dockerfile
```

---

## 開發規範 (Development Guidelines)

### 1. 核心規則 (Core Rules)

- **video_processor.py**：實作所有影片合成與字幕疊加邏輯
- **audio_processor.py**：處理字幕到音訊 (TTS) 的相關流程
- **content_processor.py**：管理背景素材的判斷機制 (圖片/影片選擇)
- 建議使用絕對匯入 (absolute imports)

### 2. 服務開發 (Service Development)

以下為此專案中各目錄的功能重點：

1. **`app/tasks/`**
    
    - **video_tasks.py**：定義各種 Celery 任務（如生成音訊 generate_audio_segment、處理段落 process_segment 等）
    - **orchestrator.py**：協調子任務之間的流程（如 orchestrator.generate_video → 串接子任務 → 完成後呼叫 finalize_video）
2. **`app/api/`**
    
    - **routes.py**：建立 FastAPI 路由（例如 `/generate`、`/status/{task_id}`）。
    - 若需資料模型，可在此或另外新增 `models.py`。
3. **`app/config/`**
    
    - **settings.py**：儲存/管理系統參數（影片輸出路徑、字幕路徑、Celery/Redis 設定等）。
4. **Docker (docker/)**
    
    - **Dockerfile**、**docker-compose.yml**：針對容器化部署所需的設定。

### 3. 框架使用 (Framework Usage)

#### FastAPI

- 使用 FastAPI 建立 HTTP API 端點（POST `/generate`、GET `/status/{task_id}`等）。
- 採用適當的 pydantic 模型做 request/response 驗證。
- 可使用 FastAPI 的依賴注入 (Depends) 機制做認證或參數管理。

#### Celery

- 在 **celery_app.py** 中設定 Celery，指明 broker/backend（預設使用 Redis）。
- 利用 group、chain 等機制做多段落並行處理。
- 若需要更多監控，可配置 Flower 或其它 Celery 監控工具。

---

### 4. 錯誤處理 (Error Handling)

- 在 Celery 任務中加上 `max_retries`、`default_retry_delay` 等參數，或在程式中呼叫 `self.retry()` 做重試機制。
- 建議在子任務失敗時，明確記錄 `exc_info=True` 並回傳錯誤訊息至主任務，確保流程可中斷或標記失敗。
- 使用 `task_status.py` 中的 `update_task_status()`，持續更新並紀錄任務狀態，方便追蹤與除錯。

### 5. 開發流程 (Development Process)

1. 在 **core/** 實作核心邏輯 (影片、音訊處理)。
2. 在 **tasks/video_tasks.py** 中定義各子任務（生成音訊/合成段落）。
3. 在 **tasks/orchestrator.py** 中協調子任務串接（如 orchestrator.generate_video → group/chain → finalize_video）。
4. 在 **api/routes.py** 中定義 API 端點 (e.g. `/generate`, `/generate/all`, `/status/...`)