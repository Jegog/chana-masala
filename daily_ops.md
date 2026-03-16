# 運用段階に日常的に実行するコマンド順

ここでは、Windows アプリまたは Docker + JupyterLab が
D:\Data\chana-masala\data\Integrated_Reports
（Docker からは /work_dir/data_shared/chana-masala/data/Integrated_Reports）
を更新した後を想定します。bind mount ではホストの同じファイルをコンテナから見ます。

# 日常運用 A: 最新版を取り込むとき
Git の最新版だけではデータ本体は来ません。Git でコードと .dvc ファイルを更新し、その後 dvc pull で remote storage から実データを取ります。dvc pull は dvc push の逆で、remote からデータを取得するためのコマンドです。
```PowerShell
cd D:\Data\chana-masala
git pull origin main
dvc pull
```

# 日常運用 B: データを更新したとき
DVC の基本動作は、データ変更 → dvc add → dvc push → Git に .dvc を commit → git push です。
DVC 公式も、dvc push は cache から remote へデータを送るためのコマンドだと説明しています。
```PowerShell
cd D:\Data\chana-masala
git status
dvc add data\Integrated_Reports
dvc push
git add data\Integrated_Reports.dvc
git commit -m "Update Integrated_Reports"
git push origin main
```

## 覚え方
何が変わったか確認
```PowerShell
git status
```
DVC に「このデータの最新版はこれ」と教える
```PowerShell
dvc add
```
その実データを GCS に送る
```PowerShell
dvc push
```
DVC のメタ情報を GitHub に送る
```PowerShell
git add / git commit / git push
```
という順です。


# 日常運用 C: 変更状態を確認したいとき
DVC には状態確認のための dvc status があります。Git 側の状態は git status で確認します。
```PowerShell
cd D:\Data\chana-masala
git status
dvc status
```

# 日常運用 D: 間違って作業ファイルを壊したので、DVC で戻したいとき
DVC には dvc checkout があり、Git 上の .dvc や dvc.lock が指す状態に workspace を戻せます。DVC は Git と組み合わせて dvc checkout などのコマンドを提供しています。
```PowerShell
cd D:\Data\chana-masala
dvc checkout
```

# DVC 日常運用手順（Windows / chana-masala）
## 目的
`D:\Data\chana-masala` を Git + DVC で運用し、  
`data\Integrated_Reports` の更新を安全に管理する。

## 前提
・Git / DVC の初回セットアップは完了済み
・実データの場所:
```text
D:\Data\chana-masala\data\Integrated_Reports
```
・Docker / JupyterLab から見える場所:
```text
/work_dir/data_shared/chana-masala/data/Integrated_Reports
```
- Git の正本運用は Windows 上
- DVC の主要コマンドも Windows 上で実行

# 基本ルール
1. データ取得
git pull origin main
dvc pull
2. データ更新
3. データ更新をGCSに反映
dvc add
dvc push
git add
git commit
git push

# 1. 最新版を取得したいとき
## 実行コマンド
```PowerShell
cd D:\Data\chana-masala
git pull origin main
dvc pull
```

# 2. データを更新したとき
## 対象ケース
・Windows アプリで Integrated_Reports 内のファイルを更新した
・Docker + JupyterLab の Python で Integrated_Reports 内のファイルを更新した
・ファイルを追加・削除・差し替えた
・ディレクトリ内の構成が変わった

## 実行コマンド
```PowerShell
cd D:\Data\chana-masala
git status

dvc add data\Integrated_Reports
dvc push
git add data\Integrated_Reports.dvc
git commit -m "Update Integrated_Reports"
git push origin main
```

# 3. 内容を確認したいとき
## Git の状態確認
```PowerShell
cd D:\Data\chana-masala
git status
```
## DVC の状態確認
```PowerShell
cd D:\Data\chana-masala
dvc status
```


## 補足
git pull
コードや .dvc ファイルを取得する
dvc pull
GCS remote から実データを取得する

# 4. 間違って作業フォルダを壊したとき
現在の Git / DVC の状態に合わせて、workspace を戻したいとき。
```PowerShell
cd D:\Data\chana-masala
dvc checkout
```

# 5. よく使う運用パターン
## パターンA: 他人が更新した内容を取り込む
```PowerShell
cd D:\Data\chana-masala
git pull origin main
dvc pull
```

## パターンB: PDF を追加した
例
data\Integrated_Reports\13770_サカタのタネ\202505\newfile.pdf を追加
```PowerShell
cd D:\Data\chana-masala
dvc add data\Integrated_Reports
dvc push
git add data\Integrated_Reports.dvc
git commit -m "Add new PDF files to Integrated_Reports"
git push origin main
```

## パターンC: Python で CSV や PDF を更新した
```PowerShell
cd D:\Data\chana-masala
dvc add data\Integrated_Reports
dvc push
git add data\Integrated_Reports.dvc
git commit -m "Update generated files in Integrated_Reports"
git push origin main
```
# 6. GitHub Desktop を使う場合の流れ
GitHub Desktop は Git 用と考える。
## データ更新後の流れ
1. Windows アプリまたは Jupyter でファイル更新
2. PowerShell で以下を実行
```PowerShell
cd D:\Data\chana-masala
dvc add data\Integrated_Reports
dvc push
```
3. GitHub Desktop を開く
4. 変更ファイルを確認する
5. commit message を入力して Commit
6. Push origin

# 7. 初心者向けの最小コマンド
## 最新版を取り込むとき
```PowerShell
cd D:\Data\chana-masala
git pull origin main
dvc pull
```
## 更新したとき
```PowerShell
cd D:\Data\chana-masala
dvc add data\Integrated_Reports
dvc push
git add data\Integrated_Reports.dvc
git commit -m "Update Integrated_Reports"
git push origin main
```

# 8. 日常運用での注意点
1. git push だけで終わらせない
dvc push を忘れると、GitHub にメタ情報だけあり、GCS に実データがない状態になる。

2. 実データ本体を Git に直接 add しない
Git に入れるのは主に以下。
・data\Integrated_Reports.dvc
・.dvc\config
・.dvcignore
・.gitattributes
・必要なコードや設定ファイル

3. 認証情報を commit しない
以下は Git に入れない。
```PowerShell
D:\Data\credentials\my-ip-project-425607-2b1b451f2fa8.json
D:\Data\chana-masala\.dvc\config.local
```
4. DVC 設定の変更は Windows 側で行う
.dvc/config.local の編集は Windows 側に統一する。

5. 大きな変更をした後は git status を見る
予期しないファイルが変更扱いになっていないか確認する。

# 9. よくある質問
Q1. Docker 内でファイルを更新した場合も Windows 側で dvc add するのか？
はい。基本的には Windows 側で dvc add / dvc push / git commit を行う。

Q2. Integrated_Reports の中の1ファイルだけ変えた場合も dvc add data\Integrated_Reports でよいか？
はい。ディレクトリ全体を DVC で追跡しているので、それでよい。

Q3. git add . を使ってよいか？
初心者のうちは避けた方が安全。まずは必要なファイルだけ add する。

Q4. dvc push の前に git commit してもよいか？
非推奨。先に dvc push を済ませてから Git に commit / push する方が安全。

# 10. 日常運用コマンド一覧（まとめ）
## 最新版取得
```PowerShell
cd D:\Data\chana-masala
git pull origin main
dvc pull
```
## データ更新後
```PowerShell
cd D:\Data\chana-masala
git status
dvc add data\Integrated_Reports
dvc push
git add data\Integrated_Reports.dvc
git commit -m "Update Integrated_Reports"
git push origin main
```
## 最新版取得
```PowerShell
cd D:\Data\chana-masala
git pull origin main
dvc pull
```
## 状態確認
```PowerShell
cd D:\Data\chana-masala
git status
dvc status
```
## 作業ツリーを DVC の状態に戻す
```PowerShell
cd D:\Data\chana-masala
dvc checkout
```

# 11. 運用の覚え方
## 自分が更新したら
更新 → dvc add → dvc push → git commit → git push
## 他人の更新を受け取るとき
git pull → dvc pull

この順を守れば、大きな事故は起こりにくい。

