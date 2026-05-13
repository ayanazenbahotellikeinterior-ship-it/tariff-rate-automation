# タスクリスト — 粗利表 関税率更新の自動化

- ドキュメントID: TSK-20260513-001
- 最終更新: 2026-05-13
- バージョン: 0.2（サンプル検証反映）
- 関連: [requirements.html](requirements.html) / [design.html](design.html)

---

## 凡例

- 状態: `[ ]` 未着手 / `[~]` 進行中 / `[x]` 完了 / `[!]` 保留・要相談
- 各タスクには **完了条件（DoD: Definition of Done）** を1行で記載
- 「Phase 0」は AIコーディング開始ゲートを満たすための前提作業

---

## Phase 0: 実装着手前（ゲート充足のための準備）

サンプル検証（2026-05-13実施）で多くが解決済み。残るのは認証関連と最終ゲート確認のみ。

- [x] **T0-01** サンプル収集（許可証PDF / PL.xls / PO.pdf）
    - DoD: `docs/粗利表用/` に3ファイル配置済み ✅
- [x] **T0-02** PO と PL の表記差の度合い実測（要件 R-01）
    - DoD: 「表記揺れ」ではなく「表現粒度の違い」と判明。語彙辞書 + 部分一致AND で対応する方針に確定 ✅
- [x] **T0-03** 輸入許可証PDFのレイアウト確認（要件 R-02）
    - DoD: `< NN 欄>` マーカー区切りで品名/税表番号/品目番号/関税率を抽出する方針に確定 ✅
- [x] **T0-04** 粗利表のシート構造確認（要件 R-03）
    - DoD: C列=品番、AJ列=数式 `=PRODUCT(AA*, X%)`、データ3行目以降を確認 ✅
- [ ] **T0-05** Google Cloud Project の作成とサービスアカウント発行
    - DoD: `service_account.json` が `./secrets/` に配置され、`.gitignore` に追記済み
- [ ] **T0-06** 粗利表（スプシ）側でサービスアカウントのメールに**閲覧権限**を付与
    - DoD: 共有設定にサービスアカウントのメールが表示されている
- [ ] **T0-07** `vocabulary.yaml` 初版を作成（実サンプル2案件分の語彙を反映）
    - DoD: WOMEN/MEN/LADIES/MENS、bottoms→PANTS、long sleeves→TOPS、dress→DRESS、robe→ROBE、set up→SETS、Cotton twill stripe pajama の語彙が網羅されている
- [ ] **T0-08** AIコーディング開始ゲートのチェックリスト充足確認
    - DoD: CLAUDE.md「AIコーディング開始ゲート」のチェック項目が全て満たされ、ユーザーから「ゲート通過」明言を得る

---

## Phase 1: プロジェクト初期化

- [ ] **T1-01** プロジェクトディレクトリ作成（design.html 13節の構成）
    - DoD: `src/` `tests/` `inputs/` `outputs/` `logs/` `secrets/` の各空ディレクトリ作成済み
- [ ] **T1-02** Python 仮想環境セットアップ（Python 3.11+）
    - DoD: `python -m venv .venv` 完了、`requirements.txt` インストールが成功する状態
- [ ] **T1-03** `requirements.txt` 作成
    - DoD: pdfplumber / openpyxl / **xlrd**（.xls対応）/ pandas / google-api-python-client / google-auth / PyYAML / pytest が `pip install -r requirements.txt` で成功
- [ ] **T1-04** `.gitignore` 作成
    - DoD: `.venv/` `secrets/` `inputs/` `outputs/` `logs/` `__pycache__/` `*.xlsx` 一時ファイルが追跡対象外
- [ ] **T1-05** `config.yaml` 初版作成（design.html 9節のテンプレート + T0で確定した値）
    - DoD: 全項目に妥当な既定値が入っている（sheets/permit/pl/po/classify/vocabulary_path）
- [ ] **T1-06** `vocabulary.yaml` をプロジェクトルートに配置（T0-07の内容を反映）
    - DoD: gender / item / material の3セクションが揃っている

---

## Phase 2: 共通基盤の実装

- [ ] **T2-01** `src/normalize_hs.py` 実装
    - DoD: 区切り除去・頭N桁切出しの関数。`"6208210000" → "620821"`, `"6302.31-0000" → "630231"`, `"6208.21-000" → "620821"` のユニットテストが緑
- [ ] **T2-02** `src/normalize_tariff.py` 実装
    - DoD: `"S 4.5%" → ("S","4.5%")`, `"G FREE" → ("G","0%")`, `"5.4%" → ("","5.4%")` 等のユニットテストが緑
- [ ] **T2-03** `src/vocabulary.py` 実装（YAML読込 + マッピング関数）
    - DoD: `vocabulary.yaml` を読み、PO値 → トークンリスト変換が動作。ユニットテスト緑
- [ ] **T2-04** `src/config.py` 実装（YAML読込 + dataclass化 + 必須項目検証）
    - DoD: 不正なYAMLで分かりやすいエラーメッセージが出る
- [ ] **T2-05** `src/logger.py` 実装（コンソール + `./logs/run_*.log`）
    - DoD: INFOレベル以上が記録される
- [ ] **T2-06** `src/file_classifier.py` 実装（拡張子 + ファイル名パターンで種別判定）
    - DoD: サンプルフォルダで許可証/PL/POが正しく振り分けられる

---

## Phase 3: 各 reader の実装

- [ ] **T3-01** `src/permit_pdf_reader.py` 実装
    - DoD: 許可書 IS2611304.pdf から **16欄全件** の (欄, 品名, 税表番号, 品目番号, 関税率_raw, 関税率, hs_key) が抽出できる。pdfplumber テーブル抽出 + 正規表現フォールバック
- [ ] **T3-02** `src/pl_reader.py` 実装（.xls 旧形式対応）
    - DoD: PL.xls から **52データ行全件** の (description, hs_code_raw, hs_key, row_no, encoded_no) が抽出できる
- [ ] **T3-03** `src/po_reader.py` 実装（列マッピング対応・Excel優先・PDFフォールバック）
    - DoD: 新PO.pdf（cotton stripe pajama 3品番）から (hinban, item, gender, quality, material, is_setup, match_tokens) が抽出できる
- [ ] **T3-04** `src/sheet_reader.py` 実装（C列・3行目以降）
    - DoD: 実際の粗利表からヘッダ + 全行を行順保持で取得でき、C列の品番が抽出できる（サービスアカウント認証経由）

---

## Phase 4: 突合・出力の実装

- [ ] **T4-01** `src/matcher.py` 実装（三ホップ + Set up展開 + 部分一致AND）
    - DoD: モックDFを用いたテストで以下7ケースが期待通り動く：
        - ✅ マッチ成功（通常）
        - ✅ Set up展開でHS一致
        - ❌ POに品番なし
        - ❌ PLにItem/Quality該当なし
        - ❌ PLで複数候補ヒット
        - ❌ Set up展開でHS不一致
        - ❌ 許可証に該当HSコードなし
- [ ] **T4-02** `src/excel_writer.py` 実装（3シート: 結果/証跡/サマリ）
    - DoD: サンプル入力で出力Excelが生成され、シート1の関税率列・未マッチ理由列、シート2の証跡列、シート3のサマリが期待通り出力される
- [ ] **T4-03** `src/main.py` 実装（CLIエントリ・全体オーケストレーション）
    - DoD: `python -m src.main` でサンプル一式から中間Excelが生成され、コンソールにサマリが表示される

---

## Phase 5: 結合テスト・受け入れ確認

- [ ] **T5-01** `tests/` にユニットテストを集約し、`pytest` で全件パス
    - DoD: `pytest` の終了コードが 0
- [ ] **T5-02** 実サンプル（許可書 IS2611304.pdf + PL.xls + 新PO.pdf）での結合実行
    - DoD: 中間Excelが生成され、3品番すべてが **関税率 7.4%** で出力される（許可証 欄07・欄14 と一致）
- [ ] **T5-03** Set up展開の動作確認
    - DoD: ctstpjlblue/ctstpjlice/ctstpjmblue について、PL の TOPS+PANTS の2行を引いて HS が同じことを確認したうえで関税率が出力される。シート2「証跡」のPL DESCRIPTION 列が2行表示になっている
- [ ] **T5-04** HS正規化の動作確認（PL 10桁 ↔ 許可証 4+2+3桁 の桁数差吸収）
    - DoD: PLの `6208210000` と許可証の `6208.21-000` が同じ hs_key `620821` で照合される
- [ ] **T5-05** 関税率正規化の動作確認（FREE → 0%、S/G プレフィックス除去）
    - DoD: 許可証16欄のうち `G FREE` の欄が `0%` として、`S 9%` が `9%` として出力される
- [ ] **T5-06** 「粗利表AJ列の数式書き換え」を実際に試す
    - DoD: 中間Excelの関税率列を見て、粗利表AJ列の `9%` を `7.4%` に書き換える運用が滞りなくできる
- [ ] **T5-07** 所要時間の実測（受け入れ条件: 30分以下）
    - DoD: 1案件をスクリプト起動〜数式書き換え完了まで30分以内
- [ ] **T5-08** 冪等性の確認
    - DoD: 同一入力で2回実行し、出力Excelの3シートとも内容が一致

---

## Phase 6: 運用整備

- [ ] **T6-01** `README.md` 作成（セットアップ手順 / 実行手順 / トラブルシュート）
    - DoD: 初見の人がREADMEだけで一通り実行できる
- [ ] **T6-02** マッチ率の実測値を design.html 16節に追記し、NFR-03（マッチ率90%）の達成可否を判定
    - DoD: 達成 → 完了 / 未達 → 次イテレーションの課題として記載
- [ ] **T6-03** 用語集 `docs/glossary.html` に本作業で確定した用語を反映（requirements 13節）
    - DoD: 品番 / Item / Quality / Set up / DESCRIPTION / HSコード / HS正規化キー / 欄 / S・G プレフィックス / FREE / PL / PO / 輸入許可証 / 粗利表 が追加されている
- [ ] **T6-04** 関税率の **S/G プレフィックス** の正確な意味を業務確認し、証跡シートに注記
    - DoD: 推定（S=一般税率、G=協定/特恵税率）の検証結果を design.html に反映
- [ ] **T6-05** HS正規化粒度（6 vs 9 桁）の妥当性検証
    - DoD: 既定 6桁で問題ないか／9桁に上げる必要があるか、複数案件の実測結果に基づき判断

---

## Phase 2 候補（将来検討・本リリース対象外）

requirements 15節 / design 16節 と整合。本リリースでは着手しない。

- 粗利表 AJ列への **自動書き込み**（Sheets API 編集権限、`=PRODUCT(AA*, X%)` 数式生成）
- **あいまいマッチ**（語彙辞書で吸収しきれない表記揺れに対するレーベンシュタイン距離等）
- PL素材サイレントケースの **HS番号体系からの素材逆引き**（R-08）
- 過去案件のHS割当を蓄積した **自動学習**
- 案件フォルダ監視による **定期実行** / **GUI化**

---

## 進行管理メモ

- **現在地**: Phase 0 のうち T0-01〜T0-04 が完了。残るは T0-05〜T0-08（認証設定 + 辞書定義 + ゲートチェック）。
- **次の節目**: T0-08 通過 → Phase 1 着手可能。
- **ブロッカー候補**: T0-05/T0-06 の Google Cloud 設定（ユーザー作業が必要）。
