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
aaa/
├── README.md                     # プロジェクト概要
├── docs/                         # ドキュメント
│   ├── requirements_definition.md # 要件定義書
│   ├── project_plan.md           # プロジェクト計画書
│   ├── technical_specification.md # 技術仕様書
│   ├── data_specification.md     # データ仕様書
│   └── development_setup.md      # 開発環境設定書
├── snowflake/                    # Snowflake関連
│   ├── ddl/                      # DDL/スキーマ定義
│   ├── etl/                      # ETL処理
│   └── queries/                  # 分析・確認用クエリ
├── sagemaker/                    # SageMaker関連
│   ├── notebooks/                # Jupyter Notebook
│   ├── scripts/                  # 学習・推論スクリプト
│   └── models/                   # モデル定義
├── data/                         # データ関連
│   ├── raw/                      # 生データ（ABCD）
│   ├── processed/                # 前処理済みデータ
│   └── evaluation/               # 評価結果
├── config/                       # 設定ファイル
│   ├── snowflake_config.yml      # Snowflake接続設定
│   └── sagemaker_config.yml      # SageMaker設定
├── src/                          # ソースコード
│   ├── data_processing/          # データ前処理
│   ├── models/                   # モデル実装
│   └── utils/                    # ユーティリティ
├── tests/                        # テストコード
└── requirements.txt              # Python依存関係
```

## 主要タスク

1. **データ基盤構築** (Snowflake)
   - ABCDデータの取り込み・前処理
   - スキーマ設計・実装
   - 学習用・評価用データの分割

2. **ML/LLM検証** (SageMaker)
   - アクション推論モデルの実装
   - プロンプト設計・最適化
   - 精度評価

3. **成果物作成**
   - 技術検証レポート
   - 再現手順書

## 期待成果

- 対話→アクション推論の実現可能性検証
- Action State Tracking精度の定量評価
- 今後の拡張に向けた技術的知見

## 関連リンク

- [ABCD Dataset (GitHub)](https://github.com/asappresearch/abcd)
- [ABCD Paper (ACL Anthology)](https://aclanthology.org/2021.naacl-main.239/)
- [Papers with Code - ABCD](https://paperswithcode.com/dataset/abcd)

## 開発手順

詳細な開発手順については [`docs/development_setup.md`](docs/development_setup.md) を参照してください。 