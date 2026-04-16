
## 🛠️ 環境建置 (Installation)

請依照以下步驟進行安裝：

### 1. 建立並啟動 Conda 虛擬環境與安裝依賴套件
請打開終端機，建立一個專屬的虛擬環境，並安裝 PyTorch 與音訊處理必備套件：

# 建立虛擬環境 (建議 Python 3.9 或 3.10)
conda create -n beat python=3.9 -y

# 啟動虛擬環境
conda activate beat

# 安裝核心深度學習套件與音訊處理庫
pip install torch torchvision torchaudio
pip install librosa numpy pandas tqdm scipy scikit-learn
pip install transformers mirdata
pip install madmom

Project_Root/
├── datasets/             # 存放原始音檔與標註檔 (.txt / .beats / .csv)
│   ├── ballroom/
│   ├── hainsworth/
│   └── hainsworth_augment/
├── mert_features/        # 存放預先萃取好的 MERT 1024維特徵 (.npy)
├── dataset.py            # 資料加載與處理腳本
├── train.py             # 模型訓練腳本
└── generate_prediction_json.py  # 推論與評估腳本

