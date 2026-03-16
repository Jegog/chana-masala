# Chana-masala
## 概要
・Integrated Reportsとその加工データ(XML)及びメタファイルの管理
・Data Version Control (DVC)を使用、Gitでは.dvcファイルのみ管理、ファイルの実体はGoogle Cloud Service上にアップロード・管理

## 前提とするローカルファイル構成
D:\Data\
├─ chana-masala\
│  ├─ .git\
│  ├─ .dvc\
│  │  ├─ config
│  │  ├─ config.local
│  │  └─ ...
│  ├─ .dvcignore
│  ├─ .gitignore
│  ├─ .gitattributes
│  └ data\
│      ├─ .gitignore
│      ├─ Integrated_Reports.dvc
│      └─ Integrated_Reports\
├─ dvc-cache\
│  └─ chana-masala\
└─ credentials\

## 作業フロー概要
1. GCSからデータ取得
git pull origin main
dvc pull
2. データ更新
3. データ更新をGCSに反映
dvc add
dvc push
git add
git commit
git push
