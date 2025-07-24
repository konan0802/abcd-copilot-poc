# ABCD データ仕様書

## ABCDデータセット概要

### データセット基本情報

**ABCD (Action-Based Conversations Dataset)** は、カスタマーサポートの対話データセットで、エージェントが取るべきアクション情報を含む対話データを提供します。

| 項目 | 詳細 |
|------|------|
| **データセット名** | Action-Based Conversations Dataset (ABCD) |
| **提供元** | ASAPP Research |
| **データ規模** | 10,042対話、83,731ターン |
| **対話ドメイン** | カスタマーサポート（EC・サブスクリプション） |
| **言語** | 英語 |
| **ライセンス** | Creative Commons BY-SA 4.0 |
| **取得URL** | https://github.com/asappresearch/abcd |

### データ特徴

- **マルチターン対話**: 平均8.34ターン/対話
- **アクションアノテーション**: 55種類のアクションラベル
- **意図分類**: 顧客の意図情報
- **スロット情報**: アクション実行に必要なパラメータ
- **ポリシー制約**: ビジネスルールに基づく制約情報

## データ構造詳細

### ファイル構成

```
abcd_v1.1/
├── train.json          # 学習用データ (8,034対話)
├── dev.json            # 検証用データ (1,004対話) 
├── test.json           # テスト用データ (1,004対話)
├── kb.json             # ナレッジベース情報
├── action_list.txt     # アクション一覧
└── README.md           # データセット説明
```

### データフィールド詳細

#### 対話レベル (Conversation Level)

| フィールド | 型 | 説明 | 例 |
|-----------|----|----|-----|
| `conversation_id` | String | 対話の一意識別子 | "customer_service_1" |
| `scenario` | Object | 対話シナリオ情報 | {"kb_id": "A12B", "customer_info": {...}} |
| `original` | Array | 元の対話ターンリスト | [{"turn_id": 1, "speaker": "customer", ...}] |
| `delexed` | Array | 正規化済み対話ターンリスト | [{"turn_id": 1, "speaker": "customer", ...}] |

#### ターンレベル (Turn Level)

| フィールド | 型 | 説明 | 例 |
|-----------|----|----|-----|
| `turn_id` | Integer | ターン番号 (1から開始) | 1, 2, 3, ... |
| `speaker` | String | 発話者 | "customer", "agent" |
| `text` | String | 発話テキスト | "I want to change my shipping address" |
| `action` | String | エージェントのアクション | "modify_order" |
| `action_label` | String | アクションの詳細ラベル | "change_shipping_address" |
| `intent` | String | 顧客の意図 | "change_order" |
| `slots` | Object | スロット情報 | {"order_id": "ORD123", "new_address": "..."} |

#### アクション詳細

**アクションカテゴリ (5種類)**:

1. **Information Gathering** (情報収集)
   - `request_information`: 追加情報の要求
   - `verify_identity`: 本人確認
   - `check_account_status`: アカウント状態確認

2. **Order Management** (注文管理)
   - `modify_order`: 注文変更
   - `cancel_order`: 注文キャンセル
   - `track_order`: 注文追跡

3. **Account Services** (アカウントサービス)
   - `update_account`: アカウント更新
   - `reset_password`: パスワードリセット
   - `manage_subscription`: サブスクリプション管理

4. **Issue Resolution** (問題解決)
   - `troubleshoot`: トラブルシューティング
   - `process_refund`: 返金処理
   - `escalate_to_supervisor`: エスカレーション

5. **Communication** (コミュニケーション)
   - `provide_information`: 情報提供
   - `confirm_action`: アクション確認
   - `end_conversation`: 対話終了

### サンプルデータ

#### 対話例

```json
{
  "conversation_id": "abcd_customer_service_001",
  "scenario": {
    "kb_id": "KB_001",
    "customer_info": {
      "customer_id": "CUST123",
      "membership_level": "gold",
      "order_history": ["ORD001", "ORD002"]
    }
  },
  "original": [
    {
      "turn_id": 1,
      "speaker": "customer",
      "text": "Hi, I need to cancel my recent order",
      "intent": "cancel_order",
      "slots": {}
    },
    {
      "turn_id": 2, 
      "speaker": "agent",
      "text": "I can help you with that. Can you provide your order number?",
      "action": "request_information",
      "action_label": "request_order_number",
      "slots": {"required_info": ["order_number"]}
    },
    {
      "turn_id": 3,
      "speaker": "customer", 
      "text": "It's ORD12345",
      "intent": "provide_order_info",
      "slots": {"order_number": "ORD12345"}
    },
    {
      "turn_id": 4,
      "speaker": "agent",
      "text": "Let me check that order for you. I can see it was placed yesterday and hasn't shipped yet. I can cancel it for you.",
      "action": "cancel_order",
      "action_label": "process_cancellation", 
      "slots": {"order_id": "ORD12345", "cancellation_reason": "customer_request"}
    }
  ]
}
```

## データ前処理仕様

### 前処理パイプライン

#### 1. データクリーニング

**テキスト正規化**:
- 特殊文字・絵文字の除去
- 大文字・小文字の統一
- 数値・固有名詞の正規化（delexed版使用）

**品質フィルタリング**:
- 極端に短い/長い対話の除外
- 不完全なアノテーションの除外
- 重複対話の検出・除去

#### 2. 特徴量生成

**対話履歴の構築**:
```python
def build_dialogue_history(turns, current_turn_id, max_history=5):
    """
    対話履歴を構築（直近Nターン）
    """
    history_turns = []
    for turn in turns[:current_turn_id]:
        if len(history_turns) >= max_history:
            history_turns.pop(0)  # 古いターンを削除
        history_turns.append({
            "speaker": turn["speaker"],
            "text": turn["text"],
            "turn_id": turn["turn_id"]
        })
    return history_turns
```

**コンテキスト特徴量**:
- `turn_count`: 現在のターン数
- `dialogue_length`: 対話の総文字数
- `speaker_changes`: 話者交代回数
- `intent_sequence`: 意図の遷移履歴
- `action_sequence`: アクションの履歴

#### 3. データ分割

**分割比率**:
- **学習用**: 70% (7,030対話)
- **検証用**: 15% (1,506対話)  
- **テスト用**: 15% (1,506対話)

**分割方法**:
- シナリオベース分割（シナリオが重複しないよう配慮）
- 時系列考慮（実際の対話順序を保持）
- バランス確保（アクション分布の偏りを最小化）

```python
def split_data(conversations, train_ratio=0.7, val_ratio=0.15):
    """
    対話データを学習・検証・テストに分割
    """
    # シナリオ別グルーピング
    scenario_groups = {}
    for conv in conversations:
        scenario_id = conv["scenario"]["kb_id"]
        if scenario_id not in scenario_groups:
            scenario_groups[scenario_id] = []
        scenario_groups[scenario_id].append(conv)
    
    # 各グループから比例分割
    train_data, val_data, test_data = [], [], []
    for scenario_id, group_convs in scenario_groups.items():
        n_total = len(group_convs)
        n_train = int(n_total * train_ratio)
        n_val = int(n_total * val_ratio)
        
        train_data.extend(group_convs[:n_train])
        val_data.extend(group_convs[n_train:n_train+n_val])
        test_data.extend(group_convs[n_train+n_val:])
    
    return train_data, val_data, test_data
```

## Snowflakeデータモデル

### データベース設計

#### 命名規則
- **データベース**: `ABCD_POC`
- **スキーマ**: `CONVERSATIONS`, `FEATURES`, `MODELS`
- **テーブル接頭辞**: `ABCD_`

#### テーブル設計

#### 1. 生データテーブル (`ABCD_RAW_CONVERSATIONS`)

```sql
CREATE TABLE ABCD_RAW_CONVERSATIONS (
    conversation_id VARCHAR(100) NOT NULL,
    scenario_info VARIANT,
    total_turns INTEGER,
    created_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    data_source VARCHAR(20) DEFAULT 'original',
    PRIMARY KEY (conversation_id)
);

CREATE TABLE ABCD_RAW_TURNS (
    conversation_id VARCHAR(100) NOT NULL,
    turn_id INTEGER NOT NULL,
    speaker VARCHAR(20) NOT NULL,
    text TEXT,
    action VARCHAR(100),
    action_label VARCHAR(100), 
    intent VARCHAR(100),
    slots VARIANT,
    created_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    PRIMARY KEY (conversation_id, turn_id),
    FOREIGN KEY (conversation_id) REFERENCES ABCD_RAW_CONVERSATIONS(conversation_id)
);
```

#### 2. 前処理済みテーブル (`ABCD_PROCESSED`)

```sql
CREATE TABLE ABCD_PROCESSED_DIALOGUES (
    conversation_id VARCHAR(100) NOT NULL,
    turn_id INTEGER NOT NULL,
    dialogue_history ARRAY,
    current_utterance TEXT,
    speaker VARCHAR(20),
    target_action VARCHAR(100),
    target_action_label VARCHAR(100),
    context_features VARIANT,
    data_split VARCHAR(10), -- 'train', 'val', 'test'
    processed_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    PRIMARY KEY (conversation_id, turn_id)
);
```

#### 3. 特徴量テーブル (`ABCD_FEATURES`)

```sql
CREATE TABLE ABCD_DIALOGUE_FEATURES (
    conversation_id VARCHAR(100) NOT NULL,
    turn_id INTEGER NOT NULL,
    
    -- 基本統計特徴量
    turn_count INTEGER,
    dialogue_length INTEGER,
    average_turn_length FLOAT,
    speaker_changes INTEGER,
    
    -- 意図・アクション履歴
    intent_sequence ARRAY,
    action_sequence ARRAY,
    unique_intents_count INTEGER,
    unique_actions_count INTEGER,
    
    -- 時系列特徴量
    time_since_start INTEGER, -- ターン開始からの経過
    last_action_type VARCHAR(100),
    action_repetition_count INTEGER,
    
    -- ベクトル特徴量
    text_embedding ARRAY, -- テキスト埋め込みベクトル
    context_embedding ARRAY, -- コンテキスト埋め込み
    
    created_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    PRIMARY KEY (conversation_id, turn_id)
);
```

#### 4. 評価結果テーブル (`ABCD_EVALUATION`)

```sql
CREATE TABLE ABCD_MODEL_PREDICTIONS (
    conversation_id VARCHAR(100) NOT NULL,
    turn_id INTEGER NOT NULL,
    model_name VARCHAR(100) NOT NULL,
    model_version VARCHAR(50),
    
    predicted_action VARCHAR(100),
    prediction_confidence FLOAT,
    alternative_actions ARRAY,
    
    true_action VARCHAR(100),
    is_correct BOOLEAN,
    
    prediction_time TIMESTAMP_NTZ,
    processing_time_ms INTEGER,
    
    PRIMARY KEY (conversation_id, turn_id, model_name, model_version)
);

CREATE TABLE ABCD_EVALUATION_METRICS (
    evaluation_id VARCHAR(100) NOT NULL,
    model_name VARCHAR(100) NOT NULL,
    model_version VARCHAR(50),
    data_split VARCHAR(10), -- 'val', 'test'
    
    -- 精度指標
    accuracy FLOAT,
    macro_f1 FLOAT,
    weighted_f1 FLOAT,
    top_3_accuracy FLOAT,
    top_5_accuracy FLOAT,
    
    -- アクション別精度
    action_level_metrics VARIANT,
    
    -- 効率性指標  
    avg_processing_time_ms FLOAT,
    total_predictions INTEGER,
    
    evaluation_date TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    PRIMARY KEY (evaluation_id)
);
```

### ETL処理設計

#### データ取り込みプロセス

```sql
-- 1. JSONファイルからの生データロード
COPY INTO ABCD_RAW_CONVERSATIONS
FROM (
    SELECT 
        $1:conversation_id::VARCHAR as conversation_id,
        $1:scenario as scenario_info,
        ARRAY_SIZE($1:original) as total_turns,
        CURRENT_TIMESTAMP() as created_at,
        'original' as data_source
    FROM @abcd_stage/train.json
)
FILE_FORMAT = (TYPE = JSON);

-- 2. ターンデータの展開
INSERT INTO ABCD_RAW_TURNS
SELECT 
    conversation_id,
    turn.value:turn_id::INTEGER as turn_id,
    turn.value:speaker::VARCHAR as speaker,
    turn.value:text::TEXT as text,
    turn.value:action::VARCHAR as action,
    turn.value:action_label::VARCHAR as action_label,
    turn.value:intent::VARCHAR as intent,
    turn.value:slots as slots,
    CURRENT_TIMESTAMP() as created_at
FROM ABCD_RAW_CONVERSATIONS,
LATERAL FLATTEN(input => scenario_info:original) as turn;
```

#### 前処理プロセス

```sql
-- 対話履歴の構築
CREATE OR REPLACE FUNCTION build_dialogue_history(conv_id VARCHAR, curr_turn INTEGER)
RETURNS ARRAY
LANGUAGE SQL
AS $$
    SELECT ARRAY_AGG(
        OBJECT_CONSTRUCT(
            'speaker', speaker,
            'text', text,
            'turn_id', turn_id
        )
    ) WITHIN GROUP (ORDER BY turn_id)
    FROM ABCD_RAW_TURNS 
    WHERE conversation_id = conv_id 
    AND turn_id < curr_turn
    AND turn_id > GREATEST(1, curr_turn - 5) -- 直近5ターン
$$;

-- 前処理済みデータの生成
INSERT INTO ABCD_PROCESSED_DIALOGUES
SELECT 
    r.conversation_id,
    r.turn_id,
    build_dialogue_history(r.conversation_id, r.turn_id) as dialogue_history,
    r.text as current_utterance,
    r.speaker,
    r.action as target_action,
    r.action_label as target_action_label,
    OBJECT_CONSTRUCT(
        'intent', r.intent,
        'slots', r.slots,
        'scenario_info', c.scenario_info
    ) as context_features,
    CASE 
        WHEN c.conversation_id IN (SELECT conversation_id FROM train_split) THEN 'train'
        WHEN c.conversation_id IN (SELECT conversation_id FROM val_split) THEN 'val'
        ELSE 'test'
    END as data_split,
    CURRENT_TIMESTAMP() as processed_at
FROM ABCD_RAW_TURNS r
JOIN ABCD_RAW_CONVERSATIONS c ON r.conversation_id = c.conversation_id
WHERE r.speaker = 'agent' -- エージェントのアクションのみ予測対象
AND r.action IS NOT NULL;
```

## データ品質管理

### 品質チェック項目

#### 1. 完全性チェック
- 必須フィールドの欠損値チェック
- アクションラベルの整合性確認
- 対話の完結性（開始・終了の確認）

#### 2. 一貫性チェック  
- speaker と action の対応関係
- intent と action の論理的整合性
- slots情報の妥当性

#### 3. 精度チェック
- アクションカテゴリの分布確認
- 対話長の異常値検出
- テキスト品質の評価

```sql
-- データ品質チェッククエリ例
WITH quality_checks AS (
    SELECT 
        conversation_id,
        COUNT(*) as total_turns,
        COUNT(CASE WHEN action IS NULL AND speaker = 'agent' THEN 1 END) as missing_actions,
        COUNT(CASE WHEN text IS NULL OR LENGTH(text) = 0 THEN 1 END) as empty_texts,
        COUNT(CASE WHEN speaker NOT IN ('customer', 'agent') THEN 1 END) as invalid_speakers
    FROM ABCD_RAW_TURNS
    GROUP BY conversation_id
)
SELECT 
    COUNT(*) as total_conversations,
    COUNT(CASE WHEN missing_actions > 0 THEN 1 END) as conversations_with_missing_actions,
    COUNT(CASE WHEN empty_texts > 0 THEN 1 END) as conversations_with_empty_texts,
    COUNT(CASE WHEN invalid_speakers > 0 THEN 1 END) as conversations_with_invalid_speakers,
    AVG(total_turns) as avg_turns_per_conversation,
    MIN(total_turns) as min_turns,
    MAX(total_turns) as max_turns
FROM quality_checks;
```

### データ監視・アラート

#### 品質指標の閾値
- **欠損率**: < 1%
- **異常対話率**: < 5%  
- **アクション分布の偏り**: Gini係数 < 0.8
- **平均対話長**: 5-15ターン（異常値: <3 or >20ターン）

#### 監視ダッシュボード項目
- 日次データ取り込み件数
- データ品質指標の推移
- アクション分布の変化
- エラー率・警告の発生状況 