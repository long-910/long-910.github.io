---
layout: post
title: "Pythonで作るCSVマージツール - 2つのCSVファイルを簡単に結合"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["python", "csv", "pandas", "cli"]
date: "2025-06-05 23:50"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2025-06-05-python-csv-merge-tool)でも公開しています。


# Pythonで作るCSVマージツール - 2つのCSVファイルを簡単に結合

## はじめに

データ分析やデータ処理の現場では、複数のCSVファイルを結合する必要がよくあります。特に、異なるソースから取得したデータを統合する際に、このような操作が必要になります。

この記事では、Pythonを使用して2つのCSVファイルを指定したキーでマージするCLIツールを作成する方法を紹介します。

## 作成するツールの機能

- 2つのCSVファイルを引数として受け取る
- マージのキーを指定可能
- 出力ファイル名を指定可能（デフォルトはoutput.csv）
- 重複する列は自動的に1つにまとめる
- エラーハンドリング機能付き

## 開発環境のセットアップ

### 1. プロジェクトの作成

```bash
mkdir py_merge_csv
cd py_merge_csv
```

### 2. 仮想環境の作成と有効化

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# または
.venv\Scripts\activate  # Windows
```

### 3. 必要なパッケージのインストール

```bash
pip install pandas
```

## コードの実装

### 1. メインスクリプトの作成

`merge_csv.py`を作成し、以下のコードを実装します：

```python
#!/usr/bin/env python3
import argparse
import pandas as pd
import sys

def merge_csv_files(file1, file2, key, output_file='output.csv'):
    try:
        # CSVファイルを読み込む
        df1 = pd.read_csv(file1)
        df2 = pd.read_csv(file2)

        # キーが両方のデータフレームに存在するか確認
        if key not in df1.columns or key not in df2.columns:
            print(f"エラー: 指定されたキー '{key}' が両方のCSVファイルに存在しません。")
            sys.exit(1)

        # データフレームをマージ
        merged_df = pd.merge(df1, df2, on=key, how='outer')

        # 結果をCSVファイルに出力
        merged_df.to_csv(output_file, index=False,  encoding="utf-8-sig")
        print(f"マージが完了しました。出力ファイル: {output_file}")

    except FileNotFoundError as e:
        print(f"エラー: ファイルが見つかりません - {e}")
        sys.exit(1)
    except pd.errors.EmptyDataError:
        print("エラー: 空のCSVファイルが指定されています。")
        sys.exit(1)
    except Exception as e:
        print(f"エラーが発生しました: {e}")
        sys.exit(1)

def main():
    parser = argparse.ArgumentParser(description='2つのCSVファイルをマージします。')
    parser.add_argument('file1', help='1つ目のCSVファイルのパス')
    parser.add_argument('file2', help='2つ目のCSVファイルのパス')
    parser.add_argument('key', help='マージに使用するキー（列名）')
    parser.add_argument('-o', '--output', default='output.csv',
                      help='出力ファイル名（デフォルト: output.csv）')

    args = parser.parse_args()
    merge_csv_files(args.file1, args.file2, args.key, args.output)

if __name__ == '__main__':
    main()
```

### 2. テスト用データの準備

テスト用のCSVファイルを作成します。

`test_data/customers.csv`:
```csv
customer_id,name,email,phone
1,山田太郎,yamada@example.com,090-1234-5678
2,鈴木花子,suzuki@example.com,090-2345-6789
3,佐藤次郎,sato@example.com,090-3456-7890
```

`test_data/orders.csv`:
```csv
customer_id,order_id,product,amount,order_date
1,ORD001,ノートパソコン,150000,2024-03-01
2,ORD003,キーボード,8000,2024-03-01
3,ORD004,モニター,30000,2024-03-03
```

## 使用方法

### 基本的な使い方

```bash
python merge_csv.py test_data/customers.csv test_data/orders.csv customer_id -o merged_result.csv
```

### 実行結果

マージされた`merged_result.csv`には以下のような結果が出力されます：

```csv
customer_id,name,email,phone,order_id,product,amount,order_date
1,山田太郎,yamada@example.com,090-1234-5678,ORD001,ノートパソコン,150000,2024-03-01
2,鈴木花子,suzuki@example.com,090-2345-6789,ORD003,キーボード,8000,2024-03-01
3,佐藤次郎,sato@example.com,090-3456-7890,ORD004,モニター,30000,2024-03-03
```

## コードの解説

### 1. エラーハンドリング

このツールでは、以下のようなエラーケースに対応しています：

- ファイルが見つからない場合
- 空のCSVファイルが指定された場合
- 指定されたキーが存在しない場合
- その他の予期せぬエラー

### 2. pandasのmerge関数

`pd.merge()`関数を使用して、2つのデータフレームを結合しています。主なパラメータは：

- `on`: 結合のキーとなる列名
- `how`: 結合方法（'outer'は外部結合で、両方のデータフレームのすべての行を保持）

## 発展的な機能の追加

このツールは以下のような機能を追加することで、さらに便利にすることができます：

1. 複数のキーでの結合
2. 結合方法の選択（内部結合、外部結合、左結合、右結合）
3. 特定の列のみを選択して出力
4. 大きなファイルのチャンク処理
5. 進捗表示機能

## まとめ

この記事では、Pythonとpandasを使用してCSVファイルをマージするCLIツールを作成しました。このツールは：

- シンプルな使い方
- 堅牢なエラーハンドリング
- 柔軟な出力オプション

を備えており、データ処理の現場で役立つツールとなっています。

## 参考リンク

- [pandas公式ドキュメント](https://pandas.pydata.org/docs/)
- [Python公式ドキュメント](https://docs.python.org/ja/3/)
- [プロジェクトのGitHubリポジトリ](https://github.com/long-910/py_merge_csv) 
