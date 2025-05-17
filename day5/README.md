# day5 演習用ディレクトリ

第 5 回「MLOps」に関する演習用のディレクトリです。

# 演習の目的

本演習コンテンツでは、技術的な用語や仕組み（例：高度な実験管理や複雑なパイプライン処理など）の詳細な理解を目的とするのではなく、モデルの管理や AI システムを継続的に取り扱うための方法を実際に体験することを主眼としています。

この体験を通じて、AI を継続的に扱うための仕組みや考え方を学び、実践的なスキルへの理解を深めることを目的とします。

# 演習概要

主な演習の流れは以下のようになっています。

1. 機械学習モデルの実験管理とパイプライン
2. 整合性テスト
3. CI(継続的インテクレーション)

本演習では、GPU を使用しないため、実行環境として「Google Colab」を前提としません。ご自身の PC 等で Python の実行環境を用意していただき、演習を行なっていただいて問題ありません。

# 事前準備

GitHub を使用した演習となるため、GitHub の「fork」、「clone」、「pull request」などの操作を行います。  
GitHub をご自身の演習環境上で使用できるようにしておいてください。

「fork」、「clone」したスクリプトに対して git コマンドで構成管理を行います。  
また、講師の環境では GitHub CLI を使用する場合があります。  
必須ではありませんが、GitHub CLI を使えるようにしておくと GitHub の操作が楽に行える場合があります。

- GitHub CLI について  
  [https://docs.github.com/ja/github-cli/github-cli/about-github-cli](https://docs.github.com/ja/github-cli/github-cli/about-github-cli)

## 環境を用意できない方向け

「Google Colab」上でも演習を行うことは可能です。
演習内で MLflow というソフトウェアを使い UI を表示する際に、Google Colab 上では ngrok を使用することになります。

GitHub を「Google Colab」上で使用できるようにするのに加え、day1 の演習で使用した ngrok を使用して MLflow の UI を表示しましょう。

### 参考情報

- Google Colab 上で GitHub を使う  
   [https://zenn.dev/smiyawaki0820/articles/e346ca8b522356](https://zenn.dev/smiyawaki0820/articles/e346ca8b522356)
- Google Colab 上で MLflow UI を使う  
   [https://fight-tsk.blogspot.com/2021/07/mlflow-ui-google-colaboratory.html](https://fight-tsk.blogspot.com/2021/07/mlflow-ui-google-colaboratory.html)

# 演習内容

演習は大きく 3 つのパートに分かれています。

- 演習 1: 機械学習モデルの実験管理とパイプライン
- 演習 2: 整合性テスト
- 演習 3: CI(継続的インテクレーション)

演習 1 と演習 2 は、day5 フォルダ直下の「演習 1」と「演習 2」フォルダを使用します。

演習 3 は、リポジトリ直下の「.github」フォルダと、day5 フォルダ直下の「演習 3」フォルダを使用します。

演習に必要なライブラリは、day5 フォルダ直下の「requirements.txt」ファイルに記載しています。各自の演習環境にインストールしてください。

## 演習 1: 機械学習モデルの実験管理とパイプライン

### ゴール

- scikit-learn + MLflow + kedro を使用して、学習 → 評価 → モデル保存までのパイプラインを構築。
- パイプラインを動かす。

### 演習ステップ

1. **データ準備**

   - Titanic データを使用
   - データの取得 → 学習用・評価用に分割

2. **学習スクリプトの実装**

   - ランダムフォレストによる学習処理
   - 評価処理（accuracy）を関数化して構成

3. **モデル保存**

   - MLflow を用いて学習済みモデルをトラッキング・保存

4. **オプション**

   - MLflow UI による可視化
   - パラメータ変更や再実行による再現性の確認・テスト

5. **パイプライン化**
   - kedro を使って処理を Node 化して組み立て

#### 演習 1 で使用する主なコマンド

```bash
cd 演習1

python main.py
mlflow ui

python pipeline.py
```

---

## 演習 2: 整合性テスト

### ゴール

- データの整合性チェック
- モデルの動作テスト

### 演習ステップ

1. **データテスト**

   - pytest + great_expectations を使用
   - データの型・欠損・値範囲を検証

2. **フォーマットチェック**
   - black, flake8, などで静的コードチェックを実施

#### 演習 2 で使用する主なコマンド

```bash
cd 演習2

python main.py
pytest main.py

black main.py
```

## 演習 3: CI(継続的インテクレーション)

### ゴール

- Python コードの静的検査・ユニットテストを含む CI 構築

1. **CI ツール導入**

   - GitHub Actions を使用
   - `.github/workflows/test.yml` を作成

1. **CI 結果確認**
   - プルリクエスト時に自動でチェックを実行

#### 演習 3 で使用する主なコマンド

GitHub CLI を使用した場合のプルリクエストの流れ

```bash
git branch develop
git checkout develop
gh repo set-default
gh pr create
```

# 宿題の関連情報

## CI でのテストケースを追加する場合のアイディアサンプル

1. **モデルテスト**

   - モデルの推論精度（再現性）や推論時間を検証

2. **差分テスト**
   - 過去バージョンと比較して性能の劣化がないか確認
   - 例: `accuracy ≥ baseline`
