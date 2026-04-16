
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


資料集準備 (Dataset Preparation)
本專案使用 Ballroom、Hainsworth (包含原始與變速增強版) 以及 HSSP 資料集進行訓練。

1. 下載資料集與特徵壓縮包
https://drive.google.com/file/d/1eKcI05qtUtcQO9Gz0YsmWWOIM6i7UVCg/view?usp=sharing
我已經將原始音檔與預先抽好的 MERT 特徵 (.npy) 打包成壓縮檔。請下載並放置於專案根目錄。

3. 解壓縮與放置
下載完成後，請將檔案解壓縮。請務必確認目錄結構如下所示：


Project_Root/
├── datasets/             # 存放原始音檔與標註檔 (.txt / .beats / .csv)
│   ├── ballroom/
│   ├── hainsworth/
│   └── hainsworth_augment/
├── mert_features/        # 存放預先萃取好的 MERT 1024維特徵 (.npy)
├── dataset.py            # 資料加載與處理腳本
├── train.py             # 模型訓練腳本
├── augment_speed.py  
├── batch_extract_mert.py  
└── generate_prediction_json.py  # 推論與評估腳本


階段一：資料前處理與特徵擴增 (Augmentation & Extraction)
為了防止神經網路「死背」特定歌曲的 BPM，並適應複雜的音樂速度變化，我們對資料進行了物理層級的擴增。

🧠 處理流程
時間變速擴增 (Time-Stretching)： 使用 librosa 對 Hainsworth 等資料集進行了不變調的速度調整（0.82x 與 1.18x）。這強迫模型學習「相對節奏比例」而非絕對時間，大幅提升了模型的速度不變性 (Tempo Invariance)。

MERT 特徵萃取： 將所有音檔統一重採樣至 24kHz，輸入 MERT-v1-330M 模型，提取高解析度 (75 FPS)、1024 維的音樂語意特徵，並儲存為 .npy 格式以加速後續訓練。

標籤平滑化 (Label Smoothing)： 將原本非 0 即 1 的硬標籤，轉換為帶有高斯擴散的軟標籤 [0.2, 0.5, 1.0, 0.5, 0.2]，引導模型學習節拍的漸進機率。

🧠 階段二：模型訓練與顯存優化 (Model Training)
我們使用客製化的 Residual BiGRU 進行訓練。為了解決長序列 (整首歌曲) 導致 GPU 顯存溢出 (OOM) 的問題，本專案實作了多項訓練優化機制。

執行訓練指令
請在終端機輸入以下指令啟動訓練：

python train2.py

核心架構與訓練機制說明
線性轉換與殘差連接 (Residual Connection)： 模型首先透過 Linear Layer 將 1024 維特徵進行轉換（隱藏層維度擴張至 512）。殘差連接將原始音色細節直接送達分類器，避免微弱的弦樂起音等物理特徵在 BiGRU 的遞迴運算中被遺忘。

動態隨機裁切 (Dynamic Random Crop)： 在 dataset.py 中，模型每次 Epoch 皆會動態隨機裁切 30 秒的音樂片段進行訓練。不僅完美解決了 VRAM 瓶頸，還確保模型具備全域的聽覺視野。

混合精度與梯度累加： 啟用 PyTorch AMP (Float16) 並設定 Gradient Accumulation Steps = 4 (等效 Batch Size = 16)，以確保在消費級 GPU 上也能穩定收斂。

🚀 階段三：推論與最終評估 (Inference)
訓練完成後，最佳權重將儲存為 best_beat_model.pth。我們將使用此權重預測測試集，並結合動態貝氏網路 (DBN) 進行後處理。

python generate_prediction_json_new.py

推論與解碼邏輯
機率峰值銳化 (Peak Sharpening)： 神經網路輸出的機率將先進行平方放大 (prob ** 2)。此舉能有效壓制雜訊，讓節拍的機率峰值更加尖銳。

DBN 後處理： 將銳化後的機率輸入 madmom 提供的 DBNBeatTrackingProcessor。透過隱馬爾可夫模型 (HMM) 的轉移矩陣，將連續的機率精確解碼為具備穩定 BPM 的節拍時間點 (Timestamps)。

參考文獻 (References)
Li, Y., Yuan, R., Zhang, G., et al. (2023). MERT: Acoustic Music Understanding Model with Large-Scale Self-supervised Training. arXiv preprint arXiv:2306.00107.

Böck, S., Korzeniowski, F., Schlüter, J., Krebs, F., & Widmer, G. (2016). madmom: a new Python Audio and Music Signal Processing Library. In Proceedings of the 24th ACM International Conference on Multimedia.

執行推論與生成預測檔
