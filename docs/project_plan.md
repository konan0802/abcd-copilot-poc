# ABCD サポート担当コパイロット PoC プロジェクト計画書

## プロジェクト概要

### 目的
ABCD (Action-Based Conversations Dataset) を活用し、対話履歴から「次に取るべきアクション」を推論するサポート担当コパイロットの技術検証を実施する。

### 開発期間
- **期間**: 2025年7月28日 ～ 2025年8月11日 (15日間・2週間)
- **工数見積**: 約75人時
  - 平日: 約11日 × 3時間 = 33時間
  - 休日: 約4日 × 8時間 = 32時間
  - **実働合計**: 約65時間

### 成功基準
1. **技術的成功基準**
   - ABCDデータセットの正常取り込み・前処理完了
   - Action State Tracking精度50%以上達成
   - エンドツーエンドでの推論パイプライン構築

2. **ビジネス的成功基準**
   - 対話→アクション推論の実現可能性確認
   - 本格開発に向けた技術的課題の明確化
   - コスト・工数見積もりの精緻化

## Work Breakdown Structure (WBS)

| Phase | Task ID | タスク名 | 期間 | 工数 | 担当 | 主要成果物 | 進捗 |
|-------|---------|----------|------|------|------|------------|------|
| **Phase 1** | **1.1** | **開発環境構築** | 7/28 | 3h | インフラ/環境構築 | `config/snowflake_config.yml`<br/>`config/sagemaker_config.yml` | ☐ |
| | **1.2** | **ABCDデータセット取得・分析** | 7/29 | 8h | データエンジニア | `data/raw/abcd_dataset/`<br/>データ分析レポート | ☐ |
| | **1.3a** | **Snowflakeスキーマ設計** | 7/30 | 8h | データエンジニア | `snowflake/ddl/abcd_schema.sql` | ☐ |
| | **1.3b** | **ETL処理実装** | 7/31-8/1 | 11h | データエンジニア | `snowflake/etl/load_abcd_data.sql` | ☐ |
| | **1.3c** | **データ品質確認** | 8/2 | 3h | データエンジニア | データ品質レポート | ☐ |
| **Phase 2** | **2.1** | **データ前処理パイプライン構築** | 8/3-8/4 | 10h | データサイエンティスト | `src/data_processing/preprocess_abcd.py`<br/>`snowflake/queries/data_split.sql` | ☐ |
| | **2.2** | **ベースラインモデル実装** | 8/5-8/6 | 12h | MLエンジニア | `sagemaker/notebooks/baseline_model.ipynb`<br/>`src/models/action_predictor.py` | ☐ |
| | **2.3** | **評価フレームワーク構築** | 8/7-8/8 | 12h | MLエンジニア | `src/utils/evaluation.py`<br/>`sagemaker/notebooks/evaluation.ipynb` | ☐ |
| **Phase 3** | **3.1** | **プロンプト最適化** | 8/9-8/10 | 11h | MLエンジニア | 最適化済みプロンプトテンプレート<br/>パフォーマンス改善レポート | ☐ |
| | **3.2** | **軽量ファインチューニング検証** | 8/10-8/11 | 8h | MLエンジニア | `sagemaker/scripts/fine_tuning.py`<br/>ファインチューニング結果レポート | ☐ |
| **Phase 4** | **4.1** | **総合評価・技術検証レポート作成** | 8/11 | 8h | 全体 | `docs/technical_validation_report.md`<br/>再現手順書 | ☐ |

 