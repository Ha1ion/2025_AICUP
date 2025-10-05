# 2025 玉山 AI 挑戰賽：警示帳戶預測專案

## 專案簡介

本專案旨在參加「2025 玉山人工智慧公開挑戰賽」，目標是根據提供的帳戶交易紀錄，建立一個高效的機器學習模型，以識別出未來可能被通報為警示的人頭帳戶。

這個專案的核心檔案是 `testv2.ipynb` 其中包含了完整的資料處理、探索性分析 (EDA)、特徵工程、模型訓練與預測流程。

## 檔案結構

請確保您的專案資料夾結構如下，所有資料檔案都必須放在名為 `data` 的子資料夾中：

```bash
aicup-project/
├── data/
│   ├── acct_transaction.csv       # 交易紀錄
│   ├── acct_alert.csv             # 警示帳戶名單 (訓練目標)
│   ├── acct_predict.csv           # 待預測帳戶名單
│   └── submission_template.csv    # 提交格式範本
│
├── test.ipynb                     # 主要的 Jupyter Notebook 程式碼
└── README.md                      # 本說明檔案
```

## 環境設定與安裝

本專案建議使用 Python 虛擬環境 (`venv`) 進行管理，以避免套件版本衝突。

1.  **建立虛擬環境** (若尚未建立):
    ```bash
    python -m venv venv
    ```

2.  **啟用虛擬環境**:
    * Windows:
        ```bash
        .\venv\Scripts\activate
        ```
    * macOS / Linux:
        ```bash
        source venv/bin/activate
        ```

3.  **安裝必要套件**:
    在啟用虛擬環境後，執行以下指令來安裝所有需要的函式庫。
    ```bash
    pip install pandas numpy scikit-learn lightgbm tqdm jupyter
    ```

## 如何執行

1.  確認您已處於**已啟用**的虛擬環境中 (終端機前方應有 `(venv)` 字樣)。
2.  在終端機中啟動 Jupyter Notebook：
    ```bash
    jupyter notebook
    ```
3.  在瀏覽器打開的頁面中，點擊並打開 `testv2.ipynb` 檔案。
4.  在 Notebook 中，從第一個儲存格開始，依序點擊 "Run" (或按 `Shift + Enter`) 執行所有儲存格。

## 程式流程簡介

`testv2.ipynb` 的執行流程如下：

1.  **資料讀取與預處理: (`load_data`)**:
    * 讀取 `data` 資料夾中的核心檔案。
    * 對帳戶 ID 進行 Label Encoding，將字串轉換為數字以利模型處理。
    * 轉換時間相關特徵，便於後續計算。
2.  **特徵工程 (`feature_engineering_by_account`)**:
    * 時間窗特徵: 針對每個帳戶，回溯計算基準日前 1, 3, 7, 14, 30 天內的交易總額、次數、平均金額、標準差、交易對象數量等。
    * 網路特徵: 計算每個帳戶的總轉出對象數 (out-degree) 和總轉入來源數 (in-degree)。
    * 行為模式特徵: 計算交易時間間隔的統計特徵，如平均值、標準差等。
4.  **樣本建立與模型訓練 (LightGBM + Optuna):**:
    * 樣本建立:
        * 將 acct_alert.csv 中的帳戶作為正樣本 (label=1)。
        * 從 acct_predict.csv（待預測名單）中隨機抽樣，建立與正樣本數量相當的負樣本 (label=0)，以訓練模型區分「警示」與「可疑」帳戶。
    * 自動超參數優化: 使用 Optuna 套件，以 AUC 為指標，自動尋找 LightGBM 模型的最佳超參數組合（如學習率、樹的深度等）。
    * 交叉驗證訓練: 使用找到的最佳參數，進行 5 折交叉驗證 (StratifiedKFold) 來訓練最終模型，確保模型的穩健性。
5.  **預測與產出**:
    * Top N 策略: 不使用傳統的機率門檻值，而是選出預測機率最高的 Top 200 個帳戶作為最終的警示帳戶預測。
    * 生成一個名為 `submission.csv` 的提交檔案，格式符合競賽要求。