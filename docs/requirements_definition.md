# ABCD サポート担当コパイロット PoC 要件書

## 目的

ABCD (Action-Based Conversations Dataset) を用い、サポート担当コパイロット（= 対話履歴から「次に取るべきアクション」を推論できるタスク指向Botの中核ロジック）を、Snowflake（DWH）＋ SageMaker（ML/LLM） 上で**技術検証（PoC）**する。

ABCD は 1万件超・55種のユーザー意図を含む、人手作成のタスク指向対話データセットで、行動（アクション）ラベルを完備しており、Action State Tracking などのタスクを想定している。これを使い、対話→アクション推論の実現可能性を検証する。

**例：** 「本人確認→解約処理→確認連絡」といった手順推論

**出典：**
- [ABCD公式](https://github.com/asappresearch/abcd)・論文
- [GitHub](https://github.com/asappresearch/abcd)
- [ACL Anthology](https://aclanthology.org/2021.naacl-main.239/)

## 要件（機能面・作業範囲）

### 1. データ基盤（Snowflake）

- ABCDの生データ取り込み・前処理・スキーマ定義（対話ID、ターン、アクション、スロット等）をSnowflake上で実施
- 学習／評価で使うための**学習用・検証用の分割テーブル（またはVIEW）**を用意
- LLM 推論時に参照する最小限の特徴量テーブル（対話履歴集約など）を作成

### 2. ML/LLM 検証（SageMaker）

#### タスク
- 次アクション推論（Action State Tracking 相当）
- （任意）システム応答文の生成

#### 手法
- まずは プロンプト設計＋API/エンドポイント推論でベースラインを確立
- 必要に応じて LoRA 等による軽量ファインチューニングを検討

#### 入出力仕様（最小）
- **入力：** 対話履歴（直近Nターン）
- **出力：** 推論アクション（JSON などの構造化形式）、（任意で応答テキスト）

#### 評価
ABCDが想定するAction State Tracking 等の正解ラベルを用いた精度確認（例：正解率）

**出典：** ABCD論文は Action State Tracking / Cascading Dialogue Success をタスクとして定義
- [ACL Anthology](https://aclanthology.org/2021.naacl-main.239/)
- [arXiv](https://arxiv.org/abs/2104.00783)

### 3. 成果物（PoCアウトプット）

- Snowflake上のスキーマ/DDL・ETL/ELT手順
- SageMaker 上での学習／推論ノートブック or スクリプト
- 簡易評価結果（指標と再現手順）
- 技術検証レポート（目的・前提・方法・結果・今後の拡張余地）

## スコープ外（今回扱わない事項）

- アプリ/UI実装（チャット画面等のフロントは不要）
- レイテンシ等の非機能要件、運用・監視、セキュリティ/権限設計、リスク対応の詳細検討
- 本番運用を想定した CI/CD・監視・モデル配信戦略 などの整備

## 参照（主要ソース）

- **GitHub:** [Action-Based Conversations Dataset (ABCD)](https://github.com/asappresearch/abcd)（10K+対話・55意図、ポリシー制約下のマルチステップ手続き）
- **論文:** Chen et al., 2021, NAACL（ABCD提案・Action State Tracking/Cascading Dialogue Success定義）
  - [ACL Anthology](https://aclanthology.org/2021.naacl-main.239/)
  - [arXiv](https://arxiv.org/abs/2104.00783)
- **Papers with Code:** [ABCD 概要](https://paperswithcode.com/dataset/abcd)（タスク指向対話/Workflow Discovery）