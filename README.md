# Dify + Xinference 部署環境設定

## 簡介
本範例示範如何在具備單張 NVIDIA 4090 顯示卡的機器上，利用 Docker 快速安裝並啟動 Dify，並整合 Xinference（搭配 vLLM 後端）、以及 Streamlit 前端。最終可部署 qwen3-8b 模型，並透過自訂的 DSL 工作流檔案運行。

## 環境需求
- **作業系統**：Ubuntu 22.04（或其他 Linux 發行版）
- **硬體**：單張 NVIDIA 4090 顯示卡、至少 32GB 以上記憶體
- **軟體**：
  - Docker (版本 ≥ 20.10)
  - Docker Compose (版本 ≥ 1.29)
  - Python (版本 ≥ 3.8)
  - pip

## 安裝與設定步驟

### 1. 下載 Dify 程式碼
```bash
# 假設尚未 clone，先將專案拉到本機
git clone https://github.com/your-org/dify.git
cd dify
````

### 2\. 啟動 Docker 服務

1.  進入 Docker 目錄：
    
    ```bash
    cd docker
    ```
    
2.  複製範例環境變數檔案，並可依需求修改：
    
    ```bash
    cp .env.example .env
    ```
    
    > **說明**：如果需要設定任何環境變數（例如資料庫連線字串、API Key 等），請編輯 `.env` 內的對應參數。
    
3.  啟動容器（背景執行）：
    
    ```bash
    docker compose up -d
    ```
    
    > 這步驟會自動建立並啟動所有 Dify 相關服務，例如後端 API、資料庫、Redis 等。  
    > 若要查看容器日誌，可執行 `docker compose logs -f`。
    

### 3\. 安裝 Python 相關套件

> 請確定系統已安裝 Python 3.8+ 以及 pip。  
> 下列指令可在虛擬環境（virtualenv / venv）中執行，或直接安裝到全域環境。

```bash
pip install "xinference[vllm]"
pip install streamlit requests folium streamlit-folium openai pandas geopy pytz
```

*   `xinference[vllm]`：安裝 Xinference 與 vLLM 相關依賴，用於效能更佳的推論。
    
*   其他套件為 Streamlit 前端、地圖繪製 (Folium)、OpenAI API 呼叫，以及常用資料處理／時區套件。
    

### 4\. 啟動 Xinference 本地伺服器

由於 Dify 需要透過 Xinference 才能載入大型模型，請在另一個終端機持續保持以下指令運行（不要關閉）：

```bash
xinference-local --host 0.0.0.0 --port 9997
```

> 此程式會開啟一個本地 API 服務，預設監聽 `0.0.0.0:9997`。  
> 確認你的防火牆／網路設定允許此埠號，並且主機可正常存取。

### 5\. 在 Xinference UI 上選擇模型

1.  在瀏覽器開啟頁面：`http://<你的主機 IP>:9997/ui`
    
2.  從下拉選單或模型清單中，選擇 `qwen3-8b`（或你先前已下載並存放的模型名稱）。
    
3.  **重要參數設定**：
    
    *   **max\_model\_len**：  
        單張 4090 顯示卡推論時，需要將 `max_model_len` 設為 `20480`。  
        具體位置可能在「參數設定」或「config」區域，視不同版本略有差異。
        
    *   其餘參數可按預設或依需求調整。
        

> 完成後，請點選「載入模型」或「Start」按鈕，確認模型載入成功且狀態顯示為「Ready」。

### 6\. 在 Dify 中設定模型並匯入 DSL 工作流

1.  打開 Dify 後端管理介面（假設在本機執行，預設為 `http://localhost:3000`，實際請依照 `.env` 中的 `DIFY_HOST`、`DIFY_PORT`）。
    
2.  進入「模型管理」頁面：
    
    *   新增一個「Xinference 模型」，填寫名稱（例如：`qwen3-8b-local`），
        
    *   設定 `endpoint` 為 `http://<你的主機 IP>:9997`、`port` 為 `9997`，並選擇剛才在 Xinference 上載入的模型名稱 `qwen3-8b`。
        
3.  確認並儲存後，可執行「測試連線」，若一切正常應顯示「連線成功」。
    
4.  匯入 DSL 工作流：
    
    *   在 Dify 前端介面，點選「工作流管理」→「匯入 DSL 檔案」。
        
    *   選擇你事先準備好的 `.dsl` 檔，例如放在 `workflows/your_workflow.dsl`。
        
    *   上傳後，系統會自動解析、建立對應的節點（Nodes）與流程。
        
    *   如有需要，可手動編輯節點參數、連線順序，確保所有步驟正確串聯。
        

### 7\. 啟動 Streamlit 前端

```bash
streamlit run ai_final_base.py
```

*   這會自動開啟一個 Streamlit 應用，預設位址通常為 `http://localhost:8501`。
    
*   進入頁面後，你應該會看到自訂介面，可選擇工作流、輸入 prompt，並查看回傳結果。
    

使用說明
----

1.  **確認 Docker 服務已啟動**：
    
    ```bash
    docker compose ps
    ```
    
    所有 Dify 相關服務（API、DB、Redis 等）應顯示為 `Up` 狀態。
    
2.  **確認 Xinference 伺服器已啟動**：
    
    ```bash
    ps aux | grep xinference-local
    ```
    
    或直接在瀏覽器打開 `http://localhost:9997/ui`，若能成功顯示 UI 即代表啟動正常。
    
3.  **Streamlit 前端使用**：
    
    *   開啟 `http://localhost:8501`（或終端機顯示的本機位址），
        
    *   選擇已匯入的工作流，填入輸入欄位，點擊執行後，系統會依序呼叫 Dify 後端，並經由 Xinference 進行模型推論，最終將回傳結果顯示在網頁上。
        

注意事項
----

*   **單卡 4090 記憶體限制**：  
    由於 qwen3-8b 模型極大，實際可用的 `max_model_len` 值要依照你的 VRAM 使用率做微調。若發現顯示「Out of Memory」或崩潰，可考慮適度降低 `max_model_len`（例如 16384、12288），或使用更小的 batch size。
    
*   **CUDA 驅動相容性**：  
    確保本機已安裝 NVIDIA 官方驅動（建議 ≥ 520 版本），並且系統已安裝對應的 CUDA Toolkit（建議 ≥ 11.8）。
    
*   **網路權限**：  
    若你在雲端機器上部署，請確認防火牆或安全組放通以下埠號：
    
    *   Docker 容器內的服務（預設 3000、6379、5432 等）
        
    *   Xinference 本地伺服器 (`9997`)
        
    *   Streamlit 前端 (`8501`)
        
*   **環境隔離**：  
    建議將 Python 套件安裝在獨立的 `venv` 或 `conda` 環境中，以免與系統其他專案相衝突。
    

常見問題（FAQ）
---------

*   **Q：我執行 `xinference-local` 時，出現 `segmentation fault` 或 GPU 無法使用？**  
    A：請確認 CUDA 驅動、NCCL、cuDNN 等元件已正確安裝，且 GPU 驅動版本與 CUDA 版本相容；必要時重新編譯 vLLM GPUs 套件或更新相依庫。
    
*   **Q：Dify 與 Xinference 連線異常，提示 `connection refused`？**  
    A：
    
    1.  確認 `xinference-local` 已啟動，且監聽在 `0.0.0.0:9997`。
        
    2.  檢查防火牆或 iptables 設定，確保未封鎖 9997 埠號。
        
    3.  在 Dify 「模型管理」中，`endpoint` 不應填 `localhost`，建議改成實際的 IP（例如 `http://127.0.0.1:9997` 或 `http://<你的主機 IP>:9997`）。
        
*   **Q：How to update 模型配置？**  
    A：可在 Xinference UI 中卸載舊版本，重新上傳模型檔並設定新參數；或直接在 Dify 「模型管理」中更新對應參數並儲存。
    

參考連結
----

*   [Dify 官方 GitHub](https://github.com/your-org/dify)
    
*   [Xinference Project](https://github.com/your-org/xinference)
    
*   vLLM 官方文件
    
*   [Streamlit 官方網站](https://streamlit.io/)
    

* * *

> **最後提醒：**  
> 每次重啟主機或容器後，請先重新啟動 Docker 容器 (`docker compose up -d`)，再啟動 `xinference-local`，最後才啟動 Streamlit。保持此順序，才能確保各個服務正常互通。祝你部署順利，Happy Coding！

