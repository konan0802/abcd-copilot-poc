# 開発環境設定書

## 環境概要

ABCD サポート担当コパイロット PoC の開発環境は以下のコンポーネントで構成されます：

- **ローカル開発環境**: Python, Jupyter Notebook
- **クラウドデータ基盤**: Snowflake
- **機械学習基盤**: Amazon SageMaker
- **バージョン管理**: Git

## 前提条件

### システム要件

| 項目 | 要件 |
|------|------|
| **OS** | macOS 12+, Ubuntu 20.04+, Windows 10+ |
| **Python** | 3.9.0 以上 |
| **メモリ** | 8GB 以上推奨 |
| **ストレージ** | 20GB 以上の空き容量 |
| **ネットワーク** | インターネット接続必須 |

### アカウント・権限

必要なアカウントと権限を事前に準備してください：

#### 1. AWSアカウント
```bash
# 必要なサービス
- Amazon SageMaker (ノートブックインスタンス・学習・推論)
- IAM (権限管理)
- S3 (データ・モデル格納)
```

**必要なIAMポリシー**:
- `AmazonSageMakerFullAccess`
- `AmazonS3FullAccess`
- `IAMReadOnlyAccess`

#### 2. Snowflakeアカウント
```bash
# 必要なロール・権限
- ACCOUNTADMIN (初期設定用)
- SYSADMIN (日常運用用)  
- データベース作成・読み書き権限
```

## ローカル環境構築

### 1. Pythonとパッケージ管理

#### Python環境のセットアップ
```bash
# pyenvを使用したPythonバージョン管理 (推奨)
curl https://pyenv.run | bash

# Python 3.11のインストール
pyenv install 3.11.5
pyenv global 3.11.5

# 仮想環境の作成
python -m venv abcd_poc_env
source abcd_poc_env/bin/activate  # macOS/Linux
# または
abcd_poc_env\Scripts\activate  # Windows
```

#### 依存関係のインストール
```bash
# リポジトリのクローン
git clone <repository-url>
cd aaa

# パッケージのインストール
pip install -r requirements.txt

# 開発用追加パッケージ
pip install -e .
```

### 2. 設定ファイルの準備

#### 環境変数設定
```bash
# .envファイルの作成
cp .env.template .env

# 以下の内容を.envに設定
cat << EOF > .env
# Snowflake設定
SNOWFLAKE_ACCOUNT=your_account
SNOWFLAKE_USER=your_username
SNOWFLAKE_PASSWORD=your_password
SNOWFLAKE_WAREHOUSE=COMPUTE_WH
SNOWFLAKE_DATABASE=ABCD_POC
SNOWFLAKE_SCHEMA=CONVERSATIONS

# AWS設定
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_DEFAULT_REGION=us-east-1

# SageMaker設定
SAGEMAKER_ROLE_ARN=arn:aws:iam::account:role/service-role/AmazonSageMaker-ExecutionRole
SAGEMAKER_BUCKET=your-sagemaker-bucket

# その他の設定
LOG_LEVEL=INFO
PROJECT_NAME=abcd_poc
EOF
```

#### 設定ファイルテンプレート

**config/snowflake_config.yml**:
```yaml
connection:
  account: ${SNOWFLAKE_ACCOUNT}
  user: ${SNOWFLAKE_USER}
  password: ${SNOWFLAKE_PASSWORD}
  warehouse: ${SNOWFLAKE_WAREHOUSE}
  database: ${SNOWFLAKE_DATABASE}
  schema: ${SNOWFLAKE_SCHEMA}

data_settings:
  raw_table_prefix: "ABCD_RAW_"
  processed_table_prefix: "ABCD_PROCESSED_"
  features_table_prefix: "ABCD_FEATURES_"
  batch_size: 1000
  max_file_size_mb: 100

logging:
  level: INFO
  format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
```

**config/sagemaker_config.yml**:
```yaml
training:
  instance_type: "ml.g4dn.xlarge"
  instance_count: 1
  max_runtime_seconds: 3600
  volume_size_gb: 30

inference:
  instance_type: "ml.m5.large"
  initial_instance_count: 1
  max_concurrent_transforms: 1

model_settings:
  max_seq_length: 512
  batch_size: 8
  learning_rate: 1e-4
  num_epochs: 3
  warmup_steps: 100

data_paths:
  s3_bucket: ${SAGEMAKER_BUCKET}
  input_prefix: "abcd-data/input"
  output_prefix: "abcd-data/output"
  model_prefix: "abcd-models"
```

### 3. Jupyter Notebook設定

#### カーネルの設定
```bash
# 仮想環境をJupyterカーネルとして登録
python -m ipykernel install --user --name=abcd_poc_env --display-name="ABCD PoC"

# Jupyter Lab の起動
jupyter lab --port=8888 --no-browser
```

#### 拡張機能のインストール (Optional)
```bash
# Jupyter拡張機能
pip install jupyterlab-git
pip install jupyterlab-lsp
pip install python-lsp-server[all]

# 拡張機能の有効化
jupyter labextension install @jupyterlab/git
```

## クラウド環境構築

### 1. Snowflake環境構築

#### データベース・スキーマの作成
```sql
-- 管理者権限で実行
USE ROLE ACCOUNTADMIN;

-- データベースの作成
CREATE DATABASE IF NOT EXISTS ABCD_POC;

-- スキーマの作成
CREATE SCHEMA IF NOT EXISTS ABCD_POC.CONVERSATIONS;
CREATE SCHEMA IF NOT EXISTS ABCD_POC.FEATURES;
CREATE SCHEMA IF NOT EXISTS ABCD_POC.MODELS;

-- ウェアハウスの作成 (必要に応じて)
CREATE WAREHOUSE IF NOT EXISTS COMPUTE_WH
    WITH WAREHOUSE_SIZE = 'SMALL'
    AUTO_SUSPEND = 60
    AUTO_RESUME = TRUE;

-- ユーザー権限の設定
GRANT USAGE ON DATABASE ABCD_POC TO ROLE SYSADMIN;
GRANT USAGE ON ALL SCHEMAS IN DATABASE ABCD_POC TO ROLE SYSADMIN;
GRANT CREATE TABLE ON ALL SCHEMAS IN DATABASE ABCD_POC TO ROLE SYSADMIN;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE ABCD_POC TO ROLE SYSADMIN;
```

#### ステージの作成（データアップロード用）
```sql
-- ファイルアップロード用ステージ
CREATE STAGE IF NOT EXISTS ABCD_POC.CONVERSATIONS.ABCD_STAGE
    FILE_FORMAT = (TYPE = JSON);

-- 設定確認
DESC STAGE ABCD_POC.CONVERSATIONS.ABCD_STAGE;
```

#### 接続テスト
```python
# Python での接続テスト
import snowflake.connector
from dotenv import load_dotenv
import os

load_dotenv()

conn = snowflake.connector.connect(
    account=os.getenv('SNOWFLAKE_ACCOUNT'),
    user=os.getenv('SNOWFLAKE_USER'),
    password=os.getenv('SNOWFLAKE_PASSWORD'),
    warehouse=os.getenv('SNOWFLAKE_WAREHOUSE'),
    database=os.getenv('SNOWFLAKE_DATABASE'),
    schema=os.getenv('SNOWFLAKE_SCHEMA')
)

cursor = conn.cursor()
cursor.execute("SELECT CURRENT_VERSION()")
result = cursor.fetchone()
print(f"Snowflake version: {result[0]}")

cursor.close()
conn.close()
```

### 2. Amazon SageMaker環境構築

#### IAMロールの作成
```bash
# AWS CLI による IAM ロール作成
aws iam create-role \
    --role-name AmazonSageMaker-ExecutionRole-abcd-poc \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "Service": "sagemaker.amazonaws.com"
                },
                "Action": "sts:AssumeRole"
            }
        ]
    }'

# 必要なポリシーをアタッチ
aws iam attach-role-policy \
    --role-name AmazonSageMaker-ExecutionRole-abcd-poc \
    --policy-arn arn:aws:iam::aws:policy/AmazonSageMakerFullAccess

aws iam attach-role-policy \
    --role-name AmazonSageMaker-ExecutionRole-abcd-poc \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

#### S3バケットの作成
```bash
# データ保存用バケットの作成
aws s3 mb s3://abcd-poc-data-$(date +%Y%m%d)

# バケット名を環境変数に設定
export SAGEMAKER_BUCKET=abcd-poc-data-$(date +%Y%m%d)
echo "SAGEMAKER_BUCKET=$SAGEMAKER_BUCKET" >> .env
```

#### SageMaker接続テスト
```python
# SageMaker 接続テスト
import boto3
import sagemaker
from sagemaker import get_execution_role

# SageMaker セッションの作成
sagemaker_session = sagemaker.Session()

# 実行ロールの取得 (SageMaker環境から実行する場合)
try:
    role = get_execution_role()
except:
    # ローカル環境から実行する場合
    role = "arn:aws:iam::YOUR_ACCOUNT:role/service-role/AmazonSageMaker-ExecutionRole-abcd-poc"

print(f"SageMaker execution role: {role}")
print(f"Default bucket: {sagemaker_session.default_bucket()}")
```

## 開発ワークフロー

### 1. データ準備

#### ABCDデータセットの取得
```bash
# データディレクトリの作成
mkdir -p data/raw/abcd_dataset

# ABCDデータセットのダウンロード
cd data/raw/abcd_dataset
wget https://github.com/asappresearch/abcd/raw/main/data/abcd_v1.1.tar.gz
tar -xzf abcd_v1.1.tar.gz

# データ構造の確認
ls -la abcd_v1.1/
head -n 5 abcd_v1.1/train.json
```

#### Snowflakeへのデータアップロード
```bash
# Snowflake CLI または Web UI を使用してデータアップロード
snowsql -c my_connection -f snowflake/etl/load_abcd_data.sql
```

### 2. 開発環境の確認

#### 環境診断スクリプトの実行
```python
# scripts/check_environment.py
import sys
import importlib
import os
from dotenv import load_dotenv

def check_python_version():
    """Python バージョンチェック"""
    version = sys.version_info
    print(f"Python version: {version.major}.{version.minor}.{version.micro}")
    return version.major >= 3 and version.minor >= 9

def check_packages():
    """必要なパッケージの確認"""
    required_packages = [
        'pandas', 'numpy', 'torch', 'transformers',
        'sagemaker', 'boto3', 'snowflake.connector',
        'jupyter', 'matplotlib', 'seaborn'
    ]
    
    missing_packages = []
    for package in required_packages:
        try:
            importlib.import_module(package.replace('.', '/'))
            print(f"✓ {package}")
        except ImportError:
            print(f"✗ {package} (missing)")
            missing_packages.append(package)
    
    return len(missing_packages) == 0

def check_env_variables():
    """環境変数の確認"""
    load_dotenv()
    
    required_vars = [
        'SNOWFLAKE_ACCOUNT', 'SNOWFLAKE_USER', 'SNOWFLAKE_PASSWORD',
        'AWS_ACCESS_KEY_ID', 'AWS_SECRET_ACCESS_KEY',
        'SAGEMAKER_ROLE_ARN', 'SAGEMAKER_BUCKET'
    ]
    
    missing_vars = []
    for var in required_vars:
        if os.getenv(var):
            print(f"✓ {var}")
        else:
            print(f"✗ {var} (not set)")
            missing_vars.append(var)
    
    return len(missing_vars) == 0

if __name__ == "__main__":
    print("=== 環境診断 ===")
    print("\n1. Python バージョン")
    python_ok = check_python_version()
    
    print("\n2. パッケージ確認")
    packages_ok = check_packages()
    
    print("\n3. 環境変数確認")
    env_ok = check_env_variables()
    
    print(f"\n=== 結果 ===")
    if python_ok and packages_ok and env_ok:
        print("✓ 全ての環境チェックが完了しました")
    else:
        print("✗ 環境に問題があります。上記のエラーを確認してください")
```

### 3. Git設定

#### .gitignoreの設定
```bash
# .gitignore
cat << EOF > .gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
abcd_poc_env/
pip-log.txt
pip-delete-this-directory.txt

# Jupyter Notebook
.ipynb_checkpoints

# 環境設定ファイル
.env
*.env

# データファイル
data/raw/
data/processed/
*.csv
*.json
*.parquet

# モデルファイル
models/
*.pkl
*.pt
*.pth

# Snowflake
*.key
*.pem

# AWS
.aws/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
logs/
*.log

# 一時ファイル
temp/
tmp/
EOF
```

#### pre-commitフックの設定 (Optional)
```bash
# pre-commit のインストール
pip install pre-commit

# .pre-commit-config.yaml の作成
cat << EOF > .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.7.0
    hooks:
      - id: black
        language_version: python3

  - repo: https://github.com/pycqa/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
        args: [--max-line-length=88, --extend-ignore=E203]

  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort
        args: [--profile=black]
EOF

# pre-commit のインストール
pre-commit install
```

## トラブルシューティング

### よくある問題と解決方法

#### 1. Snowflake接続エラー
```bash
# エラー: "250001: Failed to connect to DB. SSL connection error"
# 解決方法: SSL証明書の更新
pip install --upgrade snowflake-connector-python
```

#### 2. SageMaker権限エラー
```bash
# エラー: "An error occurred (ValidationException) when calling the CreateTrainingJob"
# 解決方法: IAMロールの権限確認
aws iam list-attached-role-policies --role-name AmazonSageMaker-ExecutionRole-abcd-poc
```

#### 3. メモリ不足エラー
```python
# データ処理時のメモリ不足
# 解決方法: チャンク処理
import pandas as pd

def process_large_dataset(file_path, chunk_size=1000):
    """大きなデータセットのチャンク処理"""
    for chunk in pd.read_json(file_path, lines=True, chunksize=chunk_size):
        # チャンクごとの処理
        processed_chunk = process_chunk(chunk)
        yield processed_chunk
```

#### 4. パッケージ競合
```bash
# パッケージ競合の解決
pip install pip-tools
pip-compile requirements.in --upgrade
pip-sync requirements.txt
```

### ログとデバッグ

#### ログ設定
```python
# src/utils/logging_config.py
import logging
import os
from datetime import datetime

def setup_logging(log_level=None):
    """ログ設定の初期化"""
    if log_level is None:
        log_level = os.getenv('LOG_LEVEL', 'INFO')
    
    # ログディレクトリの作成
    os.makedirs('logs', exist_ok=True)
    
    # ログファイル名
    log_filename = f"logs/abcd_poc_{datetime.now().strftime('%Y%m%d')}.log"
    
    # ログ設定
    logging.basicConfig(
        level=getattr(logging, log_level.upper()),
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        handlers=[
            logging.FileHandler(log_filename, encoding='utf-8'),
            logging.StreamHandler()
        ]
    )
    
    return logging.getLogger(__name__)
```

## パフォーマンス最適化

### 開発効率向上のためのTips

#### 1. データサンプリング
```python
# 開発時は小さなデータセットを使用
def create_dev_dataset(full_dataset_path, sample_ratio=0.1):
    """開発用の小さなデータセットを作成"""
    import random
    
    with open(full_dataset_path, 'r') as f:
        lines = f.readlines()
    
    sample_size = int(len(lines) * sample_ratio)
    sampled_lines = random.sample(lines, sample_size)
    
    dev_path = full_dataset_path.replace('.json', '_dev.json')
    with open(dev_path, 'w') as f:
        f.writelines(sampled_lines)
    
    return dev_path
```

#### 2. キャッシュの活用
```python
# 処理結果のキャッシュ
import joblib
import os

def cached_function(cache_dir='cache'):
    """関数結果をキャッシュするデコレータ"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            os.makedirs(cache_dir, exist_ok=True)
            cache_key = f"{func.__name__}_{hash(str(args) + str(kwargs))}"
            cache_path = os.path.join(cache_dir, f"{cache_key}.pkl")
            
            if os.path.exists(cache_path):
                return joblib.load(cache_path)
            
            result = func(*args, **kwargs)
            joblib.dump(result, cache_path)
            return result
        return wrapper
    return decorator
```

#### 3. 並列処理の活用
```python
# データ処理の並列化
from multiprocessing import Pool
import pandas as pd

def parallel_process_data(data_chunks, n_processes=4):
    """データ処理の並列実行"""
    with Pool(n_processes) as pool:
        results = pool.map(process_single_chunk, data_chunks)
    return pd.concat(results, ignore_index=True)
```

このドキュメントに従って開発環境を構築し、プロジェクトを開始してください。問題が発生した場合は、トラブルシューティングセクションを参照してください。 