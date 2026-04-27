---
name: validate-policy
description: 営業パイプラインエージェントのポリシー品質検証。P2（Automated Validation）の3層テスト（構造・コンテンツ・スモーク）を手動トリガーで実行し、検証レポートを生成する。
---

# コマンド: ポリシー品質検証

## コマンド名

`validate-policy`

## 説明

営業パイプライン管理エージェントのポリシーファイル群に対して品質検証を実行する。P2（Automated Validation）の3層テストを手動トリガーで実行し、検証レポートを生成する。

## 使用方法

```
/validate-policy
/validate-policy --layer structure
/validate-policy --layer content
/validate-policy --layer smoke
/validate-policy --full
```

## オプション

| オプション | 説明 |
|-----------|------|
| `--layer structure` | 構造テスト（ファイル存在・必須フィールド・参照整合性）のみ実行 |
| `--layer content` | コンテンツテスト（Classification行・GOOD/BAD例・Pitfall引用）のみ実行 |
| `--layer smoke` | スモークテスト（フロー完全性・CP配置）のみ実行 |
| `--full` | 3層テスト全て実行（デフォルト） |

## 検証項目

### Layer 1: 構造テスト

- [ ] 全18ファイルが存在するか（surveillance/4 + assessment/4 + dispatch/5 + reporting/3 + packaging/2）
- [ ] 全ファイルに `## Purpose` セクションがあるか
- [ ] 全ファイルに `**Classification**` 行があるか
- [ ] 全ファイルに `## Completion Message` セクションがあるか
- [ ] `plugin.json` の `agents/skills/commands/rules` 参照パスが存在するか

### Layer 2: コンテンツテスト

- [ ] 全18ファイルに GOOD/BAD例が各1件以上あるか
- [ ] 停滞判定を行うファイルに多層判定（Pitfall 1対策）の記述があるか
- [ ] 権限分類ファイル（A4）にPitfall 2対策の記述があるか
- [ ] Completion Messageが日本語で記述されているか

### Layer 3: スモークテスト

- [ ] 日中フローのフェーズ遷移が完全か（S1→S2→...→R3）
- [ ] 夜間フローのフェーズ遷移が完全か（S1→R2→R3）
- [ ] CP（Checkpoint）が設計通りに配置されているか
- [ ] 繰り延べフローが各フェーズに定義されているか

## 出力

検証完了時のレポート:
```
ポリシー品質検証レポート
========================
実行日時: [ISO 8601 JST]

Layer 1 構造テスト: [PASS / FAIL]
  - ファイル存在: [18/18]
  - 必須フィールド: [PASS / FAIL（詳細）]

Layer 2 コンテンツテスト: [PASS / FAIL]
  - GOOD/BAD例: [18/18ファイルに存在]
  - Pitfall引用: [PASS / FAIL（詳細）]

Layer 3 スモークテスト: [PASS / FAIL]
  - フロー完全性: [PASS / FAIL（詳細）]

総合判定: [PASS / FAIL]
```

FAILの場合はP1（Plugin Structure Generation）へ戻るリペアループを推奨する。
