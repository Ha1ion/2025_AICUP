# 2025 玉山 AI 挑戰賽：警示帳戶預測專案

## 專案簡介

本專案旨在參加「2025 玉山人工智慧公開挑戰賽」，目標是根據提供的帳戶交易紀錄，建立一個高效的機器學習模型，以識別出未來可能被通報為警示的人頭帳戶。

這個專案的核心檔案是 `test.ipynb`，其中包含了完整的資料處理、特徵工程、模型訓練與預測流程。

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
3.  在瀏覽器打開的頁面中，點擊並打開 `test.ipynb` 檔案。
4.  在 Notebook 中，從第一個儲存格開始，依序點擊 "Run" (或按 `Shift + Enter`) 執行所有儲存格。

## 程式流程簡介

`test.ipynb` 的執行流程如下：

1.  **資料讀取 (`load_data`)**: 讀取 `data` 資料夾中的 `acct_transaction.csv`、`acct_alert.csv` 等核心檔案。
2.  **特徵工程 (`feature_engineering_by_account`)**:
    * 以「帳戶」為核心，建立分析基準日 (`date`)。
    * 針對每個帳戶，回溯計算基準日前 1, 3, 7, 14, 30 天內的交易行為，建立「時間窗特徵」。
    * 產生的特徵包含各時間窗內的交易總額、次數、平均金額、標準差、交易對象數量等。
3.  **模型訓練 (`LightGBM`)**:
    * 將警示帳戶作為正樣本 (label=1)，並從非警示帳戶中抽樣作為負樣本 (label=0)，建立完整的訓練集。
    * 將訓練集再切分為「訓練子集」與「驗證子集」。
    * 在訓練子集上訓練模型，並在驗證子集上自動尋找 F1-Score 最高的**最佳預測門檻值**。
    * 使用**全部**訓練資料與找到的最佳參數，重新訓練最終模型。
4.  **預測與產出**:
    * 使用最終模型對測試集進行預測。
    * 套用找到的**最佳門檻值**來決定最終標籤 (0 或 1)。
    * 生成一個名為 `submission_final_optimized.csv` 的提交檔案，格式符合競賽要求。