# タスクリスト — Webフロントエンド追加

- ドキュメントID: TSK-20260513-002
- 最終更新: 2026-05-13
- バージョン: 0.1（ドラフト）
- 関連: [requirements.html](requirements.html) / [design.html](design.html) / [mockup.html](mockup.html)

---

## 凡例

- 状態: `[ ]` 未着手 / `[~]` 進行中 / `[x]` 完了 / `[!]` 保留・要相談
- 各タスクには **完了条件（DoD: Definition of Done）** を1行で記載
- Phase 0 は AIコーディング開始ゲートを満たすための前提作業

---

## Phase 0: 実装着手前（ゲート充足のための準備）

- [ ] **W0-01** 既存 CLI（`.steering/20260513-tariff-rate-automation/`）の実装が完了・動作確認済みである
    - DoD: CLI 単体で3ファイル → 中間Excel が生成され、関税率 7.4% が確認できる
- [ ] **W0-02** `docs/ui-specification.html` 作成・承認
    - DoD: 6画面のワイヤフレーム・状態定義・遷移図・UI要素一覧・アクセシビリティ方針が記載されユーザー承認済み
- [ ] **W0-03** `docs/architecture.html` 作成・承認
    - DoD: 技術スタックとホスティング候補比較が記載されユーザー承認済み
- [ ] **W0-04** Google Cloud Project 作成 + OAuth クライアントID/シークレット発行
    - DoD: `GOOGLE_OAUTH_CLIENT_ID` / `GOOGLE_OAUTH_CLIENT_SECRET` が取得済み、リダイレクトURIに `/auth/callback` を登録済み
- [ ] **W0-05** 許可ドメインリストの確定
    - DoD: 利用者のメールドメインを洗い出し、`ALLOWED_DOMAINS` の値が確定
- [ ] **W0-06** ホスティング先の最終決定（または「ローカル動作のみ先行」と確定）
    - DoD: デプロイ先が決まる、または「Phase 1 はローカル動作確認のみで Phase 2 で本番化」と合意
- [ ] **W0-07** AIコーディング開始ゲートのチェックリスト充足確認
    - DoD: CLAUDE.md ゲート要件が全て満たされ、ユーザーから「ゲート通過」明言を得る

---

## Phase 1: プロジェクト基盤整備

- [ ] **W1-01** `web/` ディレクトリ作成（design.html 13節の構成）
    - DoD: `web/routes/`, `web/middlewares/`, `web/templates/`, `web/static/` の空ディレクトリ作成済み
- [ ] **W1-02** `requirements.txt` を更新（既存に追加）
    - DoD: fastapi / uvicorn / authlib / jinja2 / python-multipart / itsdangerous が `pip install` で成功
- [ ] **W1-03** `.env.example` 作成（環境変数テンプレート）
    - DoD: 6つの必須環境変数のサンプル値が記載されている
- [ ] **W1-04** `web_config.yaml` 初版作成
    - DoD: タイムアウト・同時実行数（5）・ファイルサイズ上限（10MB）・セッションTTL等が記載
- [ ] **W1-05** `Dockerfile` + `docker-compose.yml` 作成
    - DoD: `docker compose up` でローカル 8000 番でサーバが起動する
- [ ] **W1-06** `.gitignore` 更新
    - DoD: `data/` `.env` `secrets/` `__pycache__/` `web/static/dist/` が追跡対象外

---

## Phase 2: CLI 側の公開API追加

> 既存 CLI モジュールには触れず、`pipeline.py` を1ファイル追加するのみ。

- [ ] **W2-01** `src/pipeline.py` 実装（`run_pipeline()` 公開API）
    - DoD: 3ファイルパス + Sheets ID + Config + progress_callback を受け取り、`PipelineResult` を返す関数が動く
- [ ] **W2-02** `src/pipeline.py` のユニットテスト
    - DoD: フィクスチャ実サンプルで実行し、3品番すべて 7.4% が返ることを検証
- [ ] **W2-03** progress_callback の動作確認
    - DoD: 6ステップ（permit_parse / pl_parse / po_parse / sheet_fetch / matching / writing）で callback が呼ばれる

---

## Phase 3: Web バックエンド基盤

- [ ] **W3-01** `web/main.py` 実装（FastAPI アプリ初期化）
    - DoD: `uvicorn web.main:app` で起動、`GET /healthz` が 200 を返す
- [ ] **W3-02** `web/middlewares/auth.py` 実装（認証ミドルウェア）
    - DoD: 未ログインで保護ルートにアクセスすると `/login` にリダイレクトされる
- [ ] **W3-03** `web/middlewares/error.py` 実装（例外ハンドリング）
    - DoD: 例外がスタックトレース漏洩なしで `error.html` を返す
- [ ] **W3-04** `web/routes/auth.py` 実装（OAuth フロー）
    - DoD: `/login` → Google → `/auth/callback` → セッション付与 → `/` リダイレクト の完全フローが動く
- [ ] **W3-05** ドメイン制限の実装
    - DoD: 許可ドメイン外の Google アカウントでログイン試行すると 403 + error.html
- [ ] **W3-06** `web/job_store.py` 実装（ジョブ状態管理）
    - DoD: ジョブ作成・状態更新・取得・他人のジョブアクセス拒否のユニットテスト緑
- [ ] **W3-07** `web/file_store.py` 実装（ファイル保存・削除）
    - DoD: アップロードファイル保存・30日経過分の自動削除のユニットテスト緑
- [ ] **W3-08** `web/job_runner.py` 実装（非同期ジョブ実行）
    - DoD: `asyncio.Semaphore(5)` で最大5並列まで実行・progress_callback でジョブ状態更新

---

## Phase 4: API ルート実装

- [ ] **W4-01** `POST /api/upload` 実装
    - DoD: 3ファイルを受け取り、サイズ・拡張子検証、ジョブ生成、202で `job_id` を返す
- [ ] **W4-02** `GET /api/jobs/{job_id}/status` 実装
    - DoD: ジョブ状態と progress を返す。他人のジョブは 403
- [ ] **W4-03** `GET /api/jobs/{job_id}/result` 実装
    - DoD: 完了ジョブの summary + rows + trace を JSON で返す
- [ ] **W4-04** `GET /api/jobs/{job_id}/download` 実装
    - DoD: 中間Excelファイルを `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` で返す
- [ ] **W4-05** `GET /api/jobs` 実装（履歴）
    - DoD: ログインユーザーの過去30日のジョブ一覧を JSON で返す
- [ ] **W4-06** `POST /logout` 実装
    - DoD: Cookie 削除 + Google revoke → `/login` リダイレクト

---

## Phase 5: フロントエンド（テンプレート + 静的アセット）

- [ ] **W5-01** `web/static/css/app.css` 整備（mockup.html から移植）
    - DoD: モックの見た目がそのまま再現される
- [ ] **W5-02** `templates/base.html` 実装（共通レイアウト）
    - DoD: ヘッダ・フッタ・CSS読込・ユーザー名表示が動作
- [ ] **W5-03** `templates/login.html` 実装（SCR-001-login）
    - DoD: 「Googleでログイン」ボタンが `/login` へ遷移
- [ ] **W5-04** `templates/upload.html` + `static/js/upload.js` 実装（SCR-002-upload）
    - DoD: 3ドロップゾーン + 実行ボタン。ドラッグ&ドロップ・ファイル検証・FormData 送信が動く
- [ ] **W5-05** `templates/processing.html` + `static/js/processing.js` 実装（SCR-003-processing）
    - DoD: 1秒ごとに status をポーリング、progress 表示更新、完了で `/jobs/{id}` へ遷移
- [ ] **W5-06** `templates/result.html` 実装（SCR-004-result）
    - DoD: サマリ4カード + 結果テーブル + 証跡折り畳み + ダウンロードボタン
- [ ] **W5-07** `templates/history.html` + `static/js/history.js` 実装（SCR-005-history）
    - DoD: 過去ジョブ一覧テーブル、再ダウンロードリンクが動作
- [ ] **W5-08** `templates/error.html` 実装（SCR-006-error）
    - DoD: 認証拒否・処理失敗時に適切なメッセージ表示
- [ ] **W5-09** アクセシビリティ対応
    - DoD: Tab キーのみで全画面操作可能、コントラスト比 4.5:1 以上、aria-live で進捗通知

---

## Phase 6: テスト・結合確認

- [ ] **W6-01** `tests/test_web_routes.py` 実装（API テスト）
    - DoD: `fastapi.testclient` で主要エンドポイントを網羅、認証必須・他人ジョブ拒否を確認
- [ ] **W6-02** `tests/test_job_runner.py` 実装
    - DoD: 同時実行制限・状態遷移・例外時の failed 遷移を確認
- [ ] **W6-03** OAuth フローのモックテスト
    - DoD: Authlib をモックして allowed_domains 内外のメールで挙動が変わることを確認
- [ ] **W6-04** E2E 手動テスト（実サンプル）
    - DoD: 実サンプル3ファイル（許可書 IS2611304.pdf / PL.xls / 新PO.pdf）をブラウザからアップロードし、結果画面に 7.4% が表示される
- [ ] **W6-05** 受け入れ条件全項目の確認（requirements 10節）
    - DoD: 8項目すべてに ✓

---

## Phase 7: デプロイ準備

> Phase 0 の W0-06 でホスティング先が決まってから着手。

- [ ] **W7-01** `README.md` 作成（セットアップ・起動・OAuth設定手順）
    - DoD: 初見者がREADMEだけでローカル起動できる
- [ ] **W7-02** HTTPS 設定（ホスティング先依存）
    - DoD: 本番URLでHTTPSアクセス可能
- [ ] **W7-03** 環境変数の本番値設定（Cloud Secret Manager / Render Env 等）
    - DoD: 本番環境で OAuth・Sheets が動作
- [ ] **W7-04** 初回デプロイ
    - DoD: 本番URLにアクセスし、ログイン → 1案件処理が完了
- [ ] **W7-05** 利用者への案内（URL・初回ログイン手順）
    - DoD: チームメンバーが自分のアカウントでログインでき、案件を処理できる

---

## Phase 2 候補（将来検討・本リリース対象外）

- 粗利表AJ列への **自動書き込み**（Sheets API 編集権限・別ステアリングで起こす）
- リアルタイム進捗の WebSocket / SSE 化（現状はポーリング）
- DB 導入（SQLite or PostgreSQL）でジョブ履歴を永続化
- 管理者ダッシュボード（ユーザーごとの利用統計）
- Slack/Email 通知（処理完了時）
- スマホ対応 UI
- ファイルアップロードのチャンク対応（大容量対応）

---

## 進行管理メモ

- **現在地**: Phase 0 の W0-01（既存 CLI 実装）が **未着手**。本ステアリングの実装は CLI 完成後に着手。
- **クリティカルパス**: CLI 実装 → UI仕様書 → architecture → OAuth設定 → Web実装
- **ブロッカー候補**: W0-04（OAuth クライアントID 発行）・W0-06（ホスティング先）はユーザー側の作業
