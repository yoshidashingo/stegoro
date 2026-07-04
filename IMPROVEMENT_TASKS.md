# stegoro 改善タスク一覧(2026-07-04)

## この文書について

- **目的**: 公開済みプラグイン stegoro の改善タスクを、AI コーディングエージェント(Codex 等)が設計・実装に着手できる詳細度で定義する。
- **前提となる現状判定**: stegoro は「実質リリース済み」。public リポ + `.claude-plugin/marketplace.json` により `/plugin marketplace add yoshidashingo/stegoro` で今すぐインストール可能。ドキュメントサイト stegoro.com(GitHub Pages、`docs/` がソース、HTTPS 有効)が稼働中。ただし git tag / GitHub Release / CI / 自動検証は一切なく、version は `0.1.0` のまま、最終コミットは 2026-04-27(約2ヶ月停止)。
- **プロダクトの性質(重要)**: このリポはソフトウェアコードを含まない。実体は **173 個の Markdown(約 37,000 行)で構成された Claude Code プラグイン**であり、`package.json` も試験用ランタイムもない。「実装」とはプロンプト・ルール文書の作成・修正を意味し、「テスト」とは構造検証と生成品質検証を意味する。
- **作業ルール(必読)**: リポルートの `CLAUDE.md` にルーティングルールがある。**このリポ自体の改善作業は `.aidlc/aws-aidlc-rules/core-workflow.md` をエントリポイントとする AI-DLC ワークフローに従うこと**(「ステアリングポリシーの生成」依頼のときだけ `skills/generate/` を使う)。`.aidlc/` 配下は上流スナップショットのため**中身を編集しない**。
- **優先度**: P0 = リリース衛生・品質担保の土台、P1 = 利用者価値と信頼性、P2 = 中期。

## リポ構成(実装時に触る主要パス)

- `skills/generate/SKILL.md` — `/stegoro:generate` のエントリポイント。5フェーズ18ステージ(Discovery→Design→Generation→Refinement→Packaging)。
- `skills/generate/rule-details/core-workflow.md`(669行)— マスターオーケストレーター。`rule-details/{common,discovery,design,generation,refinement,packaging}/` を随時参照。
- `.claude-plugin/plugin.json`(version 0.1.0)/ `marketplace.json` — プラグイン定義・マーケットプレイス登録。
- `samples/{customer-support-agent,proposal-writer,sales-pipeline-agent}/` — 生成例3種(各々完全なプラグイン構造)。
- `docs/` — stegoro.com のソース(`index.html`, `ja.html`, `CNAME`)。**ここに内部向け文書を置かない**(公開されるため)。
- `.aidlc/` — リポ改善用 AI-DLC ルール(上流スナップショット、編集禁止)。

---

## P0: リリース衛生・品質担保の土台

### ST-01: 正式リリースの整備(CHANGELOG + タグ + GitHub Release)

- **背景/現状**: marketplace 経由でインストール可能な状態で公開されているのに、git tag 0本・GitHub Release 0本・CHANGELOG なし。利用者はどの版を使っているか特定できず、更新の告知手段もない。
- **作業内容**:
  1. ルートに `CHANGELOG.md`(Keep a Changelog 形式)を新設。git 履歴(全30コミット、2026-02-10〜04-27)を遡って 0.1.0 の内容を要約(リネーム経緯: steering-policy-generator → agent-harness-generator → stegoro も記録)。
  2. バージョンポリシーを策定して `CONTRIBUTING.md` または `docs/` 外の運用文書に明記: semver 準拠、「生成ワークフローの互換性が壊れる変更 = major、フェーズ/ルール追加 = minor、文言修正 = patch」のような判断基準。
  3. リリース手順書を作成: `plugin.json` の version 更新 → CHANGELOG 更新 → タグ → GitHub Release(リリースノートは CHANGELOG から)。
  4. `v0.1.0` のタグ付けと Release 作成。**タグ push と Release 公開はオペレータ(ユーザー)承認を得てから実施**。
- **受け入れ基準**: 現行スナップショットに v0.1.0 のタグ・Release が付き、以後の変更が CHANGELOG 運用に乗る。
- **依存**: なし。最初に実施。

### ST-02: CI 導入(構造検証・リンク検証・メタデータ検証)

- **背景/現状**: `.github/` が存在せず、Markdown の壊れ・リンク切れ・plugin.json の破損がマージ前に検知されない。コード無しリポでも CI で守れる品質は多い。
- **作業内容**:
  1. `.github/workflows/ci.yml` を新規作成(トリガー: push / pull_request)。
  2. ジョブ構成:
     - **JSON 検証**: `.claude-plugin/plugin.json` と `marketplace.json`、および `samples/*/.claude-plugin/plugin.json` が有効な JSON で、必須キー(name / version / description 等)を持つことをチェック(jq などで十分)。
     - **Markdown lint**: markdownlint(設定は緩めに開始。プロンプト文書特有の長行・HTML は許容するルール設定にする)。
     - **リンク検証**: lychee 等で内部相対リンク切れを検出(外部 URL は flaky なので許容リストか offline モードで内部のみ)。
     - **ST-03 の構造 lint スクリプト**(作成後に組込み)。
  3. `docs/` の HTML は対象外でよい(GitHub Pages で目視確認)。
- **受け入れ基準**: 意図的に壊した JSON・リンク切れの PR で CI が fail する。正常時は green。
- **依存**: なし(ST-03 のステップだけ後から追加)。

### ST-03: プロンプト資産の構造 lint スクリプト

- **背景/現状**: `core-workflow.md` が `rule-details/` 配下の多数ファイルを参照する構造だが、参照先の存在・必須セクションの有無を機械検証する仕組みがない。ファイルのリネームや削除でワークフローが静かに壊れるリスクがこのリポ最大の回帰リスク。
- **作業内容**:
  1. `scripts/lint-structure.mjs`(Node 標準モジュールのみ、依存追加なし)を新規作成。検証項目:
     - `skills/generate/SKILL.md` と `rule-details/core-workflow.md` が言及するルールファイルパスがすべて実在すること(Markdown 中のパス参照を抽出して照合)。
     - 各フェーズディレクトリ(`common/discovery/design/generation/refinement/packaging`)のファイルが core-workflow から参照されていること(孤児ファイル検出。孤児は warning)。
     - `samples/*/` が最低限のプラグイン構造(`.claude-plugin/plugin.json`、`skills/<name>/SKILL.md`)を満たすこと。
     - SKILL.md 群の frontmatter(name / description)の存在。
  2. 実行方法を README ではなく開発者向け文書(CONTRIBUTING.md)に記載し、CI(ST-02)に組み込む。
  3. 初回実行で見つかった実際の壊れ(あれば)を修正する。ただし `.aidlc/` 配下は対象外(上流スナップショット)。
- **受け入れ基準**: `node scripts/lint-structure.mjs` が exit 0。参照先ファイルを1つリネームすると fail する。
- **依存**: なし。ST-02 と並行可。

### ST-04: `.aidlc/` 上流の TODO: CRITICAL(OWASP マッピング)への対応方針決定

- **背景/現状**: `.aidlc/aws-aidlc-rule-details/extensions/security/baseline/security-baseline.md:295` に OWASP Top 10 マッピング検証待ちの `TODO: CRITICAL` が残っている。上流(AWS AI-DLC ルールセット)由来のため直接編集はしない方針。
- **作業内容**:
  1. 上流リポ(awslabs の AI-DLC ルール配布元)の最新版を確認し、当該 TODO が解消済みなら上流スナップショットを更新(差分追跡を維持したまま丸ごと差し替え)。
  2. 未解消なら、OWASP Top 10(2021)と当該 security-baseline のマッピングを自前で検証し、結果を `.aidlc/` の**外**(例: `docs/` 以外の `notes/upstream-aidlc-verification.md`)に記録。stegoro の生成ワークフロー(`skills/generate/rule-details/`)側に影響する誤りが見つかった場合のみ、生成ルール側を修正。
- **受け入れ基準**: TODO への対応方針(上流更新 or 検証記録)が文書化され、生成ルール側への影響有無が明記されている。
- **依存**: なし。

---

## P1: 利用者価値と信頼性

### ST-05: 生成品質のスモークテスト(ゴールデンサンプル検証)

- **背景/現状**: 品質担保が Red Team レビュー(エージェント運用プロセス)のみで、「`/stegoro:generate` を実行したら実際に正しい構造の成果物ができるか」を確認する再現可能な手段がない。
- **作業内容**:
  1. スモークテスト手順書 `TESTING.md` を作成: 固定の入力シナリオ(例: 「経費精算チェックエージェント」のような samples に無い題材)で `/stegoro:generate` を実行し、生成物を検証する手順。
  2. 生成物の構造検証は ST-03 のスクリプトを流用できるよう、`lint-structure.mjs` に「任意のプラグインディレクトリを引数に取って構造検証するモード」を追加。
  3. 検証観点チェックリスト: 5フェーズが承認ゲート付きで進むか、生成された core-workflow が rule-details を正しく参照するか、`.claude-plugin/` を付ければインストール可能な形になるか。
  4. リリース前(ST-01 の手順書)にこのスモークテストを必須ステップとして組み込む。
- **受け入れ基準**: 手順書に従って第三者(別のエージェントセッション)が同じ検証を再現できる。
- **依存**: ST-03。

### ST-06: ドキュメントサイト(stegoro.com)の拡充

- **背景/現状**: `docs/index.html` / `ja.html` の LP のみ。インストール後に「何をどう入力すると何ができるか」を通しで学べるコンテンツがなく、samples の解説もサイト上にない。
- **作業内容**:
  1. チュートリアルページ(EN/JA): インストール → `/stegoro:generate` 実行 → Discovery での回答例 → 生成物の確認 → 生成プラグインのインストール、を1本通しで。samples のいずれかを題材にする。
  2. samples 3種の紹介ページ(それぞれの用途・生成された構造のハイライト)。
  3. FAQ(想定: 「生成が途中で止まったら?」「既存プラグインの改善に使える?」「Claude Code 以外で使える?」)。
  4. 既存 LP からのナビゲーション追加。デザインは既存 `index.html` のトーンを踏襲し、外部 CDN 依存を増やさない。
- **受け入れ基準**: stegoro.com からチュートリアル・サンプル解説・FAQ に到達でき、EN/JA 両方で読める。
- **依存**: なし。

### ST-07: フィードバック導線の整備

- **背景/現状**: issue テンプレートがなく、生成品質の問題報告に必要な情報(入力シナリオ、フェーズ、生成物)が集まる仕組みがない。プロンプト製品は利用ログが見えないため、報告品質が改善速度を決める。
- **作業内容**:
  1. `.github/ISSUE_TEMPLATE/` に3種: バグ報告(壊れた参照・手順の行き詰まり)、生成品質報告(入力シナリオ・期待と実際・該当フェーズを必須項目に)、機能要望。
  2. `SECURITY.md`(生成ポリシーの安全性問題の報告先)と `CONTRIBUTING.md`(ST-01/03 で作る運用文書群への入口)を整備。
  3. README(EN/JA)にフィードバック導線を追記。
- **受け入れ基準**: リポの Issues 画面でテンプレートが選択でき、必須項目が機能する。
- **依存**: なし。

### ST-08: サンプルの追加(ドメイン多様性の実証)

- **背景/現状**: samples は customer-support / proposal-writer / sales-pipeline の3種で、営業・サポート系に偏っている。生成スキルの汎用性を示すには毛色の違うドメインが要る。
- **作業内容**:
  1. 異なる性質のドメインを1〜2本追加(候補: コードレビューエージェントのような開発系、経理・請求チェックのような規則駆動系。ユーザーの関心領域に合わせて選定して良い)。
  2. **必ず `/stegoro:generate` の実ワークフローを通して生成し**(= ST-05 のスモークテストを兼ねる)、手書きしない。生成過程で見つかったルールの不備は `skills/generate/rule-details/` 側に還元する。
  3. 生成物は既存 samples と同じ構造・粒度で `samples/` に配置し、README のサンプル一覧を更新。
- **受け入れ基準**: 新サンプルが ST-03 の構造 lint を pass し、生成過程の学び(ルール修正)が別コミットで記録されている。
- **依存**: ST-03、ST-05。

### ST-09: v1.0 ロードマップの策定

- **背景/現状**: version 0.1.0 のまま開発が約2ヶ月停止。「何が揃ったら 1.0 か」の定義がなく、改善の終着点が不明。
- **作業内容**: `ROADMAP.md` を作成し、1.0 の定義(例: ST-01〜ST-08 完了 + スモークテストの定常運用 + 利用実績 N 件)と、その先の検討事項(下記 ST-10 など)を優先度付きで列挙。本書(IMPROVEMENT_TASKS.md)のタスク ID と対応付ける。
- **受け入れ基準**: 1.0 リリース判断が個人の感覚でなくチェックリストでできる状態。
- **依存**: 他タスクの完了状況を反映するため最後半で実施(骨子作成は先行可)。

---

## P2: 中期

### ST-10: 他ハーネス(Codex CLI 等)対応の検討

- **背景/現状**: 生成物・生成スキルとも Claude Code(SKILL.md / .claude-plugin)前提。Codex CLI 等の他エージェントハーネスで使いたいニーズが潜在的にある。
- **作業内容(調査タスク)**: 生成物の Claude Code 依存点を棚卸し(プラグイン構造、スラッシュコマンド、frontmatter、承認ゲートの表現)、他ハーネスへの移植に必要な変換層を設計メモとしてまとめる。**実装はこの調査結果をユーザーがレビューしてから判断**。
- **受け入れ基準**: 依存点の一覧と、対応する場合の工数感・アーキテクチャ案が文書化されている。
- **依存**: ST-09(ロードマップ上の位置づけを先に決める)。

---

## 推奨着手順

1. ST-01(リリース衛生)→ ST-02(CI)→ ST-03(構造 lint)… 土台
2. ST-04(上流 TODO)… 独立、いつでも
3. ST-05(スモークテスト)→ ST-08(サンプル追加、スモークテスト兼用)
4. ST-06(サイト拡充)/ ST-07(フィードバック導線)… 並行可
5. ST-09(ロードマップ)→ ST-10(他ハーネス調査)

## Codex への補足

- **`.aidlc/` 配下と `.claude-plugin/` の上流由来部分は直接編集しない**。リポ改善は CLAUDE.md のルーティングに従い `.aidlc/` ワークフローで進める。
- `docs/` は stegoro.com として公開される。内部向け文書(本書、TESTING.md、CONTRIBUTING.md 等)はルートまたは非公開ディレクトリに置く。
- プロンプト文書の修正では、既存の文体・構造(フェーズ → ステージ → 承認ゲートの粒度、見出し階層)を踏襲する。1ファイルだけ浮いた形式にしない。
- コミットは Conventional Commits。タグ作成・Release 公開・push はユーザー確認を取ってから。
