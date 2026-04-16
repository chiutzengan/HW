
##  Installation

###  建立並啟動 Conda 虛擬環境與安裝依賴套件
```
conda create -n beat python=3.9 -y

conda activate beat

cd 61447065S/

pip install -r requirements.txt
```

### 資料集準備 (Dataset Preparation)
使用 Ballroom、Hainsworth (包含原始與變速增強版)資料集進行訓練。

1. 下載資料集與特徵壓縮包
https://drive.google.com/file/d/1eKcI05qtUtcQO9Gz0YsmWWOIM6i7UVCg/view?usp=sharing

2. 解壓縮與放置
下載完成後，請將檔案解壓縮。目錄結構如下所示：
```
61447065S.zip/
├── training_datasets/             # 存放音檔與標註檔 (.txt / .beats)
│   ├── ballroom/
│   ├── hainsworth/
│   └── hainsworth_augment/
├── mert_features/        # 存放預先提取好的 MERT 1024維特徵 (.npy)
├── dataset.py            # 資料加載與處理腳本
├── train.py             # 模型訓練腳本
├── augment_speed.py  
├── batch_extract_mert_ALL.py  
├── eval_json.py  
└── generate_prediction_json_mert.py  # 推論腳本
```

將hainsworth資料集變速，不改變音高的情況下，將原始音檔縮放速度
```
 python augment_speed.py 
```
特徵提取(一次處理 Ballroom、Hainsworth 及增強資料集的特徵轉換)
```
 python batch_extract_mert_ALL.py  
```
模型微調訓練
```
 python train.py 
```
推論與生成預測結果到`result.json`
```
 python generate_prediction_json.py result.json
```
Evaluation 計算`result.json`分數
```
 python eval_json.py  result.json
```

References
Li, Y., Yuan, R., Zhang, G., et al. (2023). MERT: Acoustic Music Understanding Model with Large-Scale Self-supervised Training. arXiv preprint arXiv:2306.00107.

Böck, S., Korzeniowski, F., Schlüter, J., Krebs, F., & Widmer, G. (2016). madmom: a new Python Audio and Music Signal Processing Library. In Proceedings of the 24th ACM International Conference on Multimedia.

