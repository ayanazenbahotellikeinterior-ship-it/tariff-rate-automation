# 関税率自動化ツール（tariff-rate-automation）

輸入案件の **粗利表（Googleスプレッドシート）の関税率更新** を自動化するツール。
複数のファイル（輸入許可証PDF・PO・PL）を横断して関税率を割り出し、現状 **1週間** かかる手作業を **30分以下** に短縮することを目標としている。

> 📌 **本リポジトリは Private（社内利用）です。** 機密情報（取引先・金額等）は含まれませんが、業務固有の用語・スキーマを含むため社外共有しないでください。

---

## 📊 ステータス

| Phase | 内容 | 状態 |
|---|---|---|
| 0 | ドキュメント整備（要件・設計・タスクリスト） | ✅ 完了 |
| 1 | CLI 実装（バックエンド・3ホップ突合・中間Excel生成） | ⏳ 着手準備中 |
| 1.5 | Web フロントエンド（FastAPI + Google OAuth） | 📝 設計済み・実装未着手 |
| 2 | 粗利表AJ列への自動書き込み（Sheets API編集権限） | 🔮 将来検討 |

現状は **ドキュメント駆動開発のセットアップ段階** です。コードはまだ含まれません。

---

## 🎯 業務フロー（三ホップ突合）

```
品番（粗利表C列）
   ↓
PO（Item / gender / Quality）
   ↓
PL（DESCRIPTION → HSコード）
   ↓
輸入許可証PDF（HSコード → 関税率）
   ↓
中間Excel 出力 → 人が粗利表AJ列の数式の「9%」部分を該当の率に書き換える
```

詳細は [docs/product-requirements.html](docs/product-requirements.html) を参照。

---

## 📚 ドキュメント

ブラウザでローカルファイルを直接開くか、Live Server 等で表示してください。

### 永続的ドキュメント（`docs/`）

| ファイル | 内容 |
|---|---|
| [`product-requirements.html`](docs/product-requirements.html) | プロダクト要求定義書（ビジョン / KPI / ロードマップ） |
| [`ui-specification.html`](docs/ui-specification.html) | UI仕様書（6画面のワイヤフレーム・状態・遷移） |
| [`architecture.html`](docs/architecture.html) | 技術仕様書（スタック・ホスティング比較） |
| [`development-guidelines.html`](docs/development-guidelines.html) | 開発ガイドライン（Python規約・Git・テスト） |
| [`glossary.html`](docs/glossary.html) | 用語集（全59語+英日対応） |

### 作業単位ドキュメント（`.steering/`）

| ディレクトリ | 内容 |
|---|---|
| [`20260513-tariff-rate-automation/`](.steering/20260513-tariff-rate-automation/) | CLI 開発の要求・設計・タスクリスト |
| [`20260513-add-web-frontend/`](.steering/20260513-add-web-frontend/) | Web 化の要求・設計・タスクリスト + [モックアップ](.steering/20260513-add-web-frontend/mockup.html) |

### ⚡ まず触ってみたい方へ

ブラウザで [`.steering/20260513-add-web-frontend/mockup.html`](.steering/20260513-add-web-frontend/mockup.html) を開くと、Web 版の完成イメージ（クリッカブルプロトタイプ）を体験できます。

---

## 🛠 セットアップ（実装着手後）

> ⚠️ 現状はまだ実装コードがないため、以下は **将来のセットアップ手順の予定** です。

### 前提

- Python 3.11 以上
- Git
- Google Cloud Project（OAuth クライアントID・サービスアカウント）

### インストール

```bash
git clone https://github.com/<owner>/tariff-rate-automation.git
cd tariff-rate-automation

# 仮想環境作成
python -m venv .venv
.\.venv\Scripts\Activate.ps1   # Windows PowerShell
# source .venv/bin/activate     # macOS / Linux

# 依存インストール
pip install -r requirements.txt
```

### 設定

```bash
# 環境変数テンプレートをコピーして編集
cp .env.example .env
# .env を編集してOAuthクライアントID等を設定

# 設定ファイル（リポジトリに含まれる）
# - config.yaml : 抽出・突合ロジックの設定
# - vocabulary.yaml : 語彙辞書（PO Item → PL DESCRIPTION 語彙の対応）
```

### サービスアカウントJSON の配置

Google Cloud Console から発行した `service_account.json` を `secrets/` 配下に置く。

> ⚠️ `secrets/` および `.env` は `.gitignore` で除外されています。**絶対にコミットしないでください。**

---

## 🚀 使い方（実装着手後）

### CLI 版

```bash
# 案件ファイル一式を inputs/ に配置
inputs/
  ├─ 許可書.pdf
  ├─ PO.pdf or PO.xlsx
  └─ PL.xls or PL.xlsx

# 実行
python -m src.main

# 出力
outputs/tariff_YYYYMMDD_HHmm.xlsx
```

### Web 版（Phase 1.5）

```bash
docker compose up
# ブラウザで http://localhost:8000 を開く
# Google アカウントでログイン → ファイル投入 → 結果表示
```

---

## 📁 リポジトリ構造

```
tariff-rate-automation/
├─ CLAUDE.md                      # プロジェクトメモリ（Claude Code 用）
├─ README.md                      # このファイル
├─ .gitignore                     # 機密ファイル除外
│
├─ docs/                          # 永続的ドキュメント
│  ├─ product-requirements.html
│  ├─ ui-specification.html
│  ├─ architecture.html
│  ├─ development-guidelines.html
│  ├─ glossary.html
│  └─ assets/                     # 共通CSS・テンプレート
│
├─ .steering/                     # 作業単位ドキュメント
│  ├─ 20260513-tariff-rate-automation/   # CLI 開発
│  └─ 20260513-add-web-frontend/         # Web 化
│
├─ src/                           # CLI 実装（実装着手後に追加）
├─ web/                           # Web 層（実装着手後に追加）
├─ tests/                         # テスト
│
├─ inputs/                        # 案件ファイル投入先（.gitignore）
├─ outputs/                       # 中間Excel出力（.gitignore）
├─ secrets/                       # 認証情報（.gitignore 必須）
├─ logs/                          # 実行ログ（.gitignore）
│
├─ config.yaml                    # ロジック設定
├─ vocabulary.yaml                # 語彙辞書
├─ web_config.yaml                # Web 設定（Phase 1.5）
├─ requirements.txt               # Python 依存
├─ Dockerfile                     # コンテナ定義（Phase 1.5）
└─ docker-compose.yml             # ローカル起動（Phase 1.5）
```

---

## 🤝 コラボレーション

### このリポジトリに参加するには

リポジトリ管理者（Owner）に GitHub ユーザー名を伝え、招待を受けてください。
招待されると `Settings → Collaborators` から参加できます。

### 開発の進め方

CLAUDE.md（プロジェクトメモリ）に **ドキュメント駆動開発の方針** が定義されています。要点：

1. **作るものをはっきりさせてから AI のコーディングを始める**（UI仕様書を含む）
2. ドキュメント変更時は **1ファイルずつ承認** を経て進める
3. AIコーディング開始ゲートを通過するまで実装に着手しない

詳細は [`CLAUDE.md`](CLAUDE.md) を参照。

---

## 🔒 セキュリティ・機密情報の扱い

- **取引先名・金額・実品番・実HSコード** などを含む実データは **`docs/粗利表用/`（gitignore対象）** に配置し、絶対にコミットしないでください。
- Google OAuth クライアントID・サービスアカウントJSON も **`secrets/` / `.env`（gitignore対象）** に置きます。
- 万一機密情報をコミットしてしまった場合は、即座にトークン・キーを **無効化（rotate）** してください。git 履歴からの除去はその後です。

---

## 📜 ライセンス

社内ツール。社外配布・利用は不可。
（必要に応じて将来 LICENSE ファイルを追加）

---

## 📧 連絡先

リポジトリ Owner にご連絡ください。
