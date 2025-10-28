# 🔧 後端實現指南 - PDF 解鎖保留文字層

## 📋 目錄

1. [技術限制說明](#技術限制說明)
2. [推薦解決方案](#推薦解決方案)
3. [完整實現步驟](#完整實現步驟)
4. [部署選項](#部署選項)
5. [安全性考量](#安全性考量)

---

## 技術限制說明

### 為什麼純前端無法保留文字層？

目前的 PDF 解鎖工具使用純前端技術，存在以下限制：

- ✅ **PDF.js**：可以解密並渲染加密的 PDF，但只能輸出為 Canvas（影像）
- ❌ **pdf-lib**：可以複製未加密 PDF 的頁面並保留文字層，但無法處理加密的 PDF
- ⚠️ **結論**：加密 PDF 解鎖在純前端環境下必然會失去文字層

### 解決方案

需要使用後端處理來實現「解鎖加密 PDF 並保留文字層」的功能。

---

## 推薦解決方案

### 方案比較

| 方案 | 難度 | 成本 | 保留品質 | 推薦指數 |
|------|------|------|----------|----------|
| Python + pikepdf | ⭐⭐ | 免費-$7/月 | ✅ 100% | ⭐⭐⭐⭐⭐ |
| Node.js + QPDF | ⭐⭐⭐ | 免費-$7/月 | ✅ 100% | ⭐⭐⭐⭐ |
| Java + PDFBox | ⭐⭐⭐⭐ | 自建 | ✅ 100% | ⭐⭐⭐ |
| Serverless（AWS/CF） | ⭐⭐⭐ | 按用量 | ✅ 100% | ⭐⭐⭐⭐ |

**推薦：Python FastAPI + pikepdf** - 簡單、可靠、免費部署

---

## 完整實現步驟

### 第一步：建立後端專案

#### 1.1 建立專案目錄

```bash
mkdir pdf-unlock-backend
cd pdf-unlock-backend
```

#### 1.2 創建 `requirements.txt`

```txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
pikepdf==8.7.1
python-multipart==0.0.6
```

#### 1.3 創建 `main.py`

```python
from fastapi import FastAPI, File, UploadFile, Form, HTTPException
from fastapi.responses import StreamingResponse
from fastapi.middleware.cors import CORSMiddleware
import pikepdf
import io
import logging

# 設定日誌
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(
    title="PDF 解鎖 API",
    description="解除 PDF 密碼保護並保留文字層",
    version="1.0.0"
)

# CORS 設定
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 生產環境請改為你的前端網域
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 設定檔案大小限制（50MB）
MAX_FILE_SIZE = 50 * 1024 * 1024

@app.get("/")
async def root():
    return {
        "message": "PDF 解鎖 API 運行中",
        "endpoints": {
            "unlock": "/unlock-pdf (POST)",
            "health": "/health (GET)"
        }
    }

@app.get("/health")
async def health_check():
    return {"status": "healthy"}

@app.post("/unlock-pdf")
async def unlock_pdf(
    file: UploadFile = File(..., description="加密的 PDF 文件"),
    password: str = Form(..., description="PDF 密碼")
):
    """
    解除 PDF 密碼保護並保留文字層
    
    - **file**: 上傳的加密 PDF 文件
    - **password**: PDF 密碼
    
    返回：解鎖後的 PDF 文件（保留文字層）
    """
    try:
        # 檢查文件類型
        if not file.filename.lower().endswith('.pdf'):
            raise HTTPException(
                status_code=400, 
                detail="僅支援 PDF 文件"
            )
        
        # 讀取文件
        pdf_bytes = await file.read()
        
        # 檢查文件大小
        if len(pdf_bytes) > MAX_FILE_SIZE:
            raise HTTPException(
                status_code=413, 
                detail=f"文件過大，最大支援 {MAX_FILE_SIZE // 1024 // 1024}MB"
            )
        
        logger.info(f"處理文件: {file.filename}, 大小: {len(pdf_bytes)} bytes")
        
        # 使用 pikepdf 解鎖
        try:
            pdf = pikepdf.open(io.BytesIO(pdf_bytes), password=password)
        except pikepdf.PasswordError:
            raise HTTPException(
                status_code=401, 
                detail="密碼錯誤，請確認後重試"
            )
        except Exception as e:
            logger.error(f"PDF 解析錯誤: {str(e)}")
            raise HTTPException(
                status_code=400, 
                detail=f"PDF 文件解析失敗: {str(e)}"
            )
        
        # 保存到記憶體（移除密碼保護）
        output = io.BytesIO()
        pdf.save(output)
        pdf.close()
        output.seek(0)
        
        logger.info(f"成功解鎖: {file.filename}")
        
        # 生成下載文件名
        original_name = file.filename.rsplit('.', 1)[0]
        unlocked_filename = f"{original_name}_unlocked.pdf"
        
        # 返回解鎖後的 PDF
        return StreamingResponse(
            output,
            media_type="application/pdf",
            headers={
                "Content-Disposition": f'attachment; filename="{unlocked_filename}"',
                "X-Preserved-Text-Layer": "true"
            }
        )
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"未預期的錯誤: {str(e)}")
        raise HTTPException(
            status_code=500, 
            detail=f"伺服器錯誤: {str(e)}"
        )

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 第二步：本地測試

```bash
# 安裝依賴
pip install -r requirements.txt

# 啟動伺服器
python main.py

# 伺服器會在 http://localhost:8000 運行
# 訪問 http://localhost:8000/docs 查看 API 文檔
```

### 第三步：修改前端（整合後端 API）

在 `pdf_unlock_converter.html` 中添加後端模式選項：

```javascript
// 在 unlockPDF 函數開頭添加
async function unlockPDF() {
    const password = passwordInput.value.trim();
    
    if (!password || !currentFile) {
        showError('請輸入密碼並選擇文件！');
        return;
    }

    // 檢查是否使用後端 API
    const useBackend = true; // 設為 true 啟用後端模式
    const backendUrl = 'https://your-api.com/unlock-pdf'; // 替換為你的後端 URL

    if (useBackend) {
        // 使用後端 API（保留文字層）
        try {
            progressSection.style.display = 'block';
            passwordSection.style.display = 'none';
            progressText.textContent = '正在上傳並解鎖（保留文字層）...';
            progressFill.style.width = '30%';
            
            const formData = new FormData();
            formData.append('file', currentFile);
            formData.append('password', password);
            
            const response = await fetch(backendUrl, {
                method: 'POST',
                body: formData
            });
            
            progressFill.style.width = '70%';
            
            if (!response.ok) {
                const error = await response.json();
                throw new Error(error.detail || '解鎖失敗');
            }
            
            unlockedPdfData = await response.blob();
            
            progressFill.style.width = '100%';
            progressText.textContent = '✅ 解鎖完成（已保留文字層）';
            
            // 獲取頁數（簡單估算）
            const arrayBuffer = await unlockedPdfData.arrayBuffer();
            const pdf = await pdfjsLib.getDocument(arrayBuffer).promise;
            const totalPages = pdf.numPages;
            
            await new Promise(resolve => setTimeout(resolve, 300));
            displayResults(totalPages, true); // 傳遞 preservedText=true
            
        } catch (error) {
            console.error('後端解鎖錯誤:', error);
            showError('解鎖失敗：' + error.message);
            progressSection.style.display = 'none';
            passwordSection.style.display = 'block';
        }
        return;
    }
    
    // 原有的前端處理邏輯...
}

// 修改 displayResults 函數
function displayResults(totalPages, preservedText = false) {
    // ... 原有代碼 ...
    
    const modeText = preservedText 
        ? '✨ 保留文字層（可選取與搜尋）'
        : '🎨 印刷級品質 (300 DPI | PNG 無損)';
    
    // 更新 fileInfo 顯示...
}
```

---

## 部署選項

### 選項 1：Render.com（推薦，免費）

**1. 創建 `render.yaml`：**

```yaml
services:
  - type: web
    name: pdf-unlock-api
    env: python
    buildCommand: pip install -r requirements.txt
    startCommand: uvicorn main:app --host 0.0.0.0 --port $PORT
    healthCheckPath: /health
```

**2. 部署步驟：**
1. 推送代碼到 GitHub
2. 到 [Render.com](https://render.com) 註冊
3. 連接 GitHub 倉庫
4. 選擇 Web Service
5. 自動部署完成

**免費額度：** 750 小時/月

---

### 選項 2：Railway.app（推薦，免費）

**1. 創建 `railway.toml`（可選）：**

```toml
[build]
builder = "nixpacks"

[deploy]
startCommand = "uvicorn main:app --host 0.0.0.0 --port $PORT"
healthcheckPath = "/health"
```

**2. 部署步驟：**
```bash
# 安裝 Railway CLI
npm i -g @railway/cli

# 登入
railway login

# 初始化專案
railway init

# 部署
railway up
```

**免費額度：** $5 使用額度/月

---

### 選項 3：Vercel（Serverless）

**創建 `vercel.json`：**

```json
{
  "builds": [
    {
      "src": "main.py",
      "use": "@vercel/python"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "main.py"
    }
  ]
}
```

**部署：**
```bash
npm i -g vercel
vercel
```

---

### 選項 4：Docker（自建伺服器）

**創建 `Dockerfile`：**

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安裝依賴
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 複製應用
COPY main.py .

# 暴露端口
EXPOSE 8000

# 啟動應用
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**建置和運行：**

```bash
# 建置映像
docker build -t pdf-unlock-api .

# 運行容器
docker run -d -p 8000:8000 pdf-unlock-api
```

**使用 Docker Compose：**

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    restart: unless-stopped
    environment:
      - MAX_FILE_SIZE=52428800
```

---

## 安全性考量

### 必須實作的安全措施

#### 1. 檔案大小限制

```python
MAX_FILE_SIZE = 50 * 1024 * 1024  # 50MB

if len(pdf_bytes) > MAX_FILE_SIZE:
    raise HTTPException(status_code=413, detail="文件過大")
```

#### 2. 速率限制

```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

@app.post("/unlock-pdf")
@limiter.limit("10/minute")  # 每分鐘最多 10 次請求
async def unlock_pdf(request: Request, ...):
    # ...
```

#### 3. CORS 白名單

```python
# 生產環境設定
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://your-frontend-domain.com",
        "https://www.your-frontend-domain.com"
    ],
    allow_credentials=True,
    allow_methods=["POST"],
    allow_headers=["*"],
)
```

#### 4. HTTPS 強制

```python
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

# 生產環境添加
if os.getenv("ENVIRONMENT") == "production":
    app.add_middleware(HTTPSRedirectMiddleware)
```

#### 5. 臨時文件清理

```python
import tempfile
import os

temp_file = None
try:
    # 處理文件...
    pass
finally:
    # 確保刪除
    if temp_file and os.path.exists(temp_file):
        os.remove(temp_file)
```

#### 6. 日誌記錄（不記錄密碼）

```python
import logging

logger = logging.getLogger(__name__)

# ✅ 正確：不記錄敏感信息
logger.info(f"處理文件: {file.filename}")

# ❌ 錯誤：不要記錄密碼
# logger.info(f"密碼: {password}")  # 永遠不要這樣做！
```

---

## 成本估算

### 免費方案（適合個人/小型專案）

| 服務 | 免費額度 | 限制 |
|------|---------|------|
| Render | 750 小時/月 | 休眠策略（15分鐘無活動） |
| Railway | $5 額度/月 | 約 500 小時 |
| Vercel | 100GB 頻寬/月 | Serverless 函數 10秒超時 |
| Heroku | 550 小時/月（已停止免費方案） | - |

### 付費方案（適合商業使用）

| 服務 | 月費 | 特點 |
|------|------|------|
| Render | $7/月起 | 始終在線，無休眠 |
| Railway | $5/月起 | 按用量計費 |
| AWS Lambda | 按請求 | 前 100萬請求免費 |
| Digital Ocean | $5/月起 | VPS，完全控制 |

---

## 常見問題

### Q1: 後端會保存我的文件嗎？

**A:** 不會。範例代碼中，所有處理都在記憶體中完成，文件不會保存到磁碟。處理完成後立即返回給用戶。

### Q2: 密碼安全嗎？

**A:** 密碼通過 HTTPS 加密傳輸，後端不記錄密碼。建議部署時啟用 HTTPS。

### Q3: 可以處理多大的文件？

**A:** 範例設定為 50MB。可根據伺服器配置調整。建議：
- 免費方案：10-50MB
- 付費方案：100MB+

### Q4: 處理速度如何？

**A:** 取決於文件大小和伺服器性能：
- 小文件（<5MB）：1-3 秒
- 中型文件（5-20MB）：3-10 秒
- 大文件（20-50MB）：10-30 秒

### Q5: 如何監控 API 狀態？

**A:** 
1. 使用 `/health` 端點
2. 設定 UptimeRobot 監控
3. 查看 Render/Railway 控制台日誌

---

## 進階優化

### 1. 添加進度追蹤（WebSocket）

```python
from fastapi import WebSocket

@app.websocket("/ws/progress")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    # 實現進度更新...
```

### 2. 批次處理

```python
@app.post("/unlock-batch")
async def unlock_batch(files: List[UploadFile], password: str):
    results = []
    for file in files:
        # 處理每個文件...
        results.append(unlocked_pdf)
    return results
```

### 3. 快取機制（Redis）

```python
import redis
import hashlib

redis_client = redis.Redis(host='localhost', port=6379)

def get_cache_key(file_bytes, password):
    return hashlib.md5(file_bytes + password.encode()).hexdigest()

# 檢查快取...
```

---

## 技術支援

### 相關資源

- [pikepdf 官方文檔](https://pikepdf.readthedocs.io/)
- [FastAPI 官方文檔](https://fastapi.tiangolo.com/)
- [Render 部署指南](https://render.com/docs)
- [Railway 部署指南](https://docs.railway.app/)

### 開源專案參考

- [pdf-unlock-service](https://github.com/example/pdf-unlock-service)（範例專案）

---

## 總結

使用後端處理可以完美解決純前端無法保留文字層的問題：

✅ **優點：**
- 完整保留 PDF 文字層、向量圖形、書籤、表單
- 處理速度快
- 支援所有類型的 PDF 加密

⚠️ **注意事項：**
- 需要額外的伺服器成本（可使用免費方案）
- 文件需要上傳到伺服器（確保 HTTPS 加密）
- 需要維護後端服務

**推薦起步方案：** Render.com 免費方案 + Python FastAPI + pikepdf

---

**作者：** CHUN TING YUEH  
**更新日期：** 2025-10-28  
**版本：** 1.0.0

