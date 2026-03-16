1. dvc_setup_windows_chana_masala.md
# DVC 初回セットアップ手順（Windows / chana-masala）
## 目的
以下の構成で、Git + DVC によるデータ管理環境を Windows 上に構築する。

```text
D:\Data\
├─ chana-masala\                    ← Git + DVC repo
│  ├─ data\
│  │  └─ Integrated_Reports\        ← DVCで版管理する実データ
│  ├─ src\
│  ├─ .dvc\
│  └─ ...
├─ dvc-cache\
│  └─ chana-masala\                 ← repo外のDVC cache
└─ credentials\
   └─ my-ip-project-425607-2b1b451f2fa8.json
```

Docker では D:\Data が /work_dir/data_shared にマウントされている前提。

## 前提
・GitHub 上に chana-masala リポジトリが作成済み
・Google Cloud Storage バケットを dvc-data-repoに格納・利用
・サービスアカウントキーをD:/Data/credentials/に格納・利用
・Windows 上で Git を正本として運用
・DVC コマンドも基本的に Windows 上で実行する
・GitHub Desktop は Git の commit / push / pull 用に使う

## 事前に用意するもの
・Git for Windows
・GitHub Desktop
・Python
・DVC

## 0. GitHub Desktop にサインイン
GitHub Desktop を起動し、GitHub アカウントにサインインしておく。

## 1. Git のグローバル設定

PowerShell で実行する。
``` powershell
git config --global user.name "Naoto Hiramatsu"
git config --global user.email "GitHubに登録しているメールアドレス"
git config --global init.defaultBranch main
git config --global core.autocrlf false
git config --global core.eol lf
git config --global core.safecrlf warn
```
設定確認:
``` powershell
git config --global --list
```

## 2. GitHub リポジトリを D:\Data に clone
### 方法A: GitHub Desktop を使う
GitHub Desktop で Clone repository
リポジトリ: Jegog/chana-masala
保存先: D:\Data

結果として以下が作成される。
D:\Data\chana-masala

### 方法B: PowerShell で clone する
cd D:\Data
git clone https://github.com/Jegog/chana-masala

## 3. フォルダ骨格を作成
dvc init は現在のディレクトリを DVC プロジェクトとして初期化します。
今回の作業場所は D:\Data\chana-masalaがターゲットとなる。
``` Powerxhell
cd D:\Data\chana-masala
mkdir data
mkdir D:\Data\dvc-cache
mkdir D:\Data\dvc-cache\chana-masala

(mkdir src)
(mkdir notebooks)
```

## 4. .gitattributes を作成

改行コードを LF に寄せるため、repo ルートに .gitattributes を作る。
DVC の Windows ガイドは、.gitattributes で LF を固定することを推奨しています。
``` powershell
Set-Content -Path .gitattributes -Value "* text=auto eol=lf"
```

## 5. Git の初期コミット
DVC 導入前の基本設定だけを保存します。git status は、Git がどのファイルを変更と認識しているかを見る基本コマンドです。GitHub Desktop でも同等の確認ができます。
``` powershell
git add .gitattributes
git commit -m "Initialize repository settings"
git push origin main
```

## 6. DVC をインストール
``` powershell
pip install "dvc[gs]"
dvc version
```

## 7. DVC を初期化
repo ルートで実行する。
dvc init を実行すると .dvc/ が作成され、これは自動で git add される。
``` powershell
cd D:\Data\chana-masala
dvc init
```
この操作により、通常は以下が作成される。
D:\Data\chana-masala\.dvc\
D:\Data\chana-masala\.dvcignore


## 8. DVC cache を repo 外に設定
``` powershell
dvc cache dir D:\Data\dvc-cache\chana-masala
dvc config cache.type copy
```

## 9. GCS remote を設定
``` powershell
dvc remote add -d gcsremote gs://dvc-data-repo/chana-masala
dvc remote modify --local gcsremote credentialpath "D:/Data/credentials/my-ip-project-xxxxxx-xxxxxxxxxx.json"
```
### 補足
.dvc/config
Git 管理対象

.dvc/config.local
ローカル専用設定。Git 管理外

## 10. 既存データを repo 内へ移動
現在の管理対象フォルダ:
D:\Data\Integrated_Reports
これを repo 内へ移す。

``` powershell
robocopy `
  D:\Data\Integrated_Reports `
  D:\Data\chana-masala\data\Integrated_Reports `
  /E /MOVE
```

移動後の実データ位置:
D:\Data\chana-masala\data\Integrated_Reports

Docker から見える位置:
/work_dir/data_shared/chana-masala/data/Integrated_Reports

## 11. DVC でデータを登録
``` powershell
cd D:\Data\chana-masala
dvc add data\Integrated_Reports
```
これにより、通常は以下が作成・更新される。
D:\Data\chana-masala\data\Integrated_Reports.dvc
D:\Data\chana-masala\data\.gitignore

## 12. DVC remote に実データを push
``` powershell
dvc push
```

## 13. Git に DVC メタ情報を commit / push
``` powershell
git add .dvc .dvcignore .gitattributes .gitignore data\Integrated_Reports.dvc data\.gitignore
git commit -m "Set up DVC and track Integrated_Reports"
git push origin main
```

## 14. 任意: Git push 時に DVC push を自動化
初心者向けには有用。
``` powershell
dvc install
```
# セットアップ後のディレクトリイメージ
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
│  ├─ data\
│  │  ├─ .gitignore
│  │  ├─ Integrated_Reports.dvc
│  │  └─ Integrated_Reports\
│  ├─ src\
│  └─ notebooks\
├─ dvc-cache\
│  └─ chana-masala\
└─ credentials\
   └─ my-ip-project-425607-2b1b451f2fa8.json

# セットアップ時の要点
・Git の repo ルートは D:\Data\chana-masala
・実データは data\Integrated_Reports に置く
・DVC cache は D:\Data\dvc-cache\chana-masala
・GCS remote は gs://dvc-data-repo/chana-masala
・GCS の認証情報は .dvc/config.local にのみ保持する
・実データ本体は Git に入れず、DVC で管理する

# 初回セットアップ用コマンド一覧（まとめ）
```powershell
git config --global user.name "Naoto Hiramatsu"
git config --global user.email "あなたのGitHubに登録しているメールアドレス"
git config --global init.defaultBranch main
git config --global core.autocrlf false
git config --global core.eol lf
git config --global core.safecrlf warn

cd D:\Data
git clone https://github.com/Jegog/chana-masala

cd D:\Data\chana-masala
mkdir data
mkdir D:\Data\dvc-cache
mkdir D:\Data\dvc-cache\chana-masala

mkdir src
mkdir notebooks
Set-Content -Path .gitattributes -Value "* text=auto eol=lf"

git add .gitattributes
git commit -m "Initialize repository settings"
git push origin main

pip install "dvc[gs]"
dvc version
dvc init

dvc cache dir D:\Data\dvc-cache\chana-masala
dvc config cache.type copy

dvc remote add -d gcsremote gs://dvc-data-repo/chana-masala
dvc remote modify --local gcsremote credentialpath "D:/Data/credentials/my-ip-project-xxxxxx-xxxxxxxxxxxxx.json"
#　実際に使用するものに書き換え

robocopy `
  D:\Data\Integrated_Reports `
  D:\Data\chana-masala\data\Integrated_Reports `
  /E /MOVE

dvc add data\Integrated_Reports
dvc push

git add .dvc .dvcignore .gitattributes .gitignore data\Integrated_Reports.dvc data\.gitignore
git commit -m "Set up DVC and track Integrated_Reports"
git push origin main

dvc install
```

# よくあるミス
1. dvc push を忘れて git push だけする
GitHub には .dvc ファイルだけ上がり、実データが GCS に上がらない。

2. Integrated_Reports を repo の外に置いたままにする
DVC の通常運用では、追跡対象は repo 内に置く方が管理しやすい。

3. サービスアカウントキーを Git に入れてしまう
認証情報は repo 外に置き、.dvc/config.local で参照する。

4. Windows 側と Docker 側の両方で .dvc/config.local を書き換える
ローカル設定が衝突しやすい。基本的に DVC の設定変更は Windows 側で行う。