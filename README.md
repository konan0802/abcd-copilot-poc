# ABCD サポート担当コパイロット PoC

## 概要

ABCD (Action-Based Conversations Dataset) を活用したサポート担当コパイロットの技術検証（PoC）プロジェクトです。<br>
対話履歴から「次に取るべきアクション」を推論するタスク指向Botの中核ロジックを、Snowflake（DWH）＋ SageMaker（ML/LLM）上で実装・評価します。

## 技術スタック

- **データ基盤**: Snowflake
- **機械学習・LLM**: Amazon SageMaker
- **データセット**: ABCD (Action-Based Conversations Dataset)

## プロジェクト構成

```
abcd-copilot-poc/
├── README.md                     # プロジェクト概要
├── requirements.txt              # Python依存関係
├── docs/                         # ドキュメント
│   ├── requirements_definition.md # 要件定義書
│   ├── project_plan.md           # プロジェクト計画書
│   ├── technical_specification.md # 技術仕様書
│   ├── data_specification.md     # データ仕様書
│   └── development_setup.md      # 開発環境設定書
├── config/                       # 設定ファイル
│   ├── snowflake_config.yml      # Snowflake接続設定
│   └── sagemaker_config.yml      # SageMaker設定
├── snowflake/                    # Snowflake関連
│   ├── ddl/                      # DDL/スキーマ定義
│   ├── etl/                      # ETL処理
│   ├── queries/                  # 分析・確認用クエリ
│   └── README.md                 # Snowflake関連の説明
├── sagemaker/                    # SageMaker関連
│   ├── notebooks/                # Jupyter Notebook
│   ├── scripts/                  # 学習・推論スクリプト
│   └── models/                   # モデル定義
├── src/                          # ソースコード
│   ├── data_processing/          # データ前処理
│   ├── models/                   # モデル実装
│   └── utils/                    # ユーティリティ
├── data/                         # データ関連
│   ├── raw/                      # 生データ（ABCD）
│   ├── processed/                # 前処理済みデータ
│   └── evaluation/               # 評価結果
└── tests/                        # テストコード
```

## 関連リンク

- [ABCD Dataset (GitHub)](https://github.com/asappresearch/abcd)
- [ABCD Paper (ACL Anthology)](https://aclanthology.org/2021.naacl-main.239/)
- [Papers with Code - ABCD](https://paperswithcode.com/dataset/abcd) 