# AI Agent Orchestration Patterns

VS Code Copilot のサブエージェント機能を活用したオーケストレーションパターン集です。

> 参考: https://code.visualstudio.com/docs/copilot/agents/subagents#_orchestration-patterns

## パターン一覧

### 1. Coordinator and Worker パターン

メインのコーディネーターエージェントがタスク全体を管理し、専門化されたワーカーエージェントにサブタスクを委譲します。

| エージェント | 役割 | ユーザー呼び出し |
|---|---|---|
| **Feature Builder** | コーディネーター。計画→設計検証→実装→レビューのサイクルを管理 | ✅ |
| **Planner** | 機能リクエストを実装タスクに分解 | ❌ (サブエージェント専用) |
| **Plan Architect** | 計画をコードベースに対して検証 | ❌ (サブエージェント専用) |
| **Implementer** | タスクに基づいてコードを実装 | ❌ (サブエージェント専用) |
| **Reviewer** | 実装をレビューしフィードバックを提供 | ❌ (サブエージェント専用) |

**使い方:** チャットで `@Feature Builder` を選択し、機能リクエストを記述します。

<details>
<summary><b>▶ 試してみよう：ステップバイステップガイド</b></summary>

#### 前提

このリポジトリを VS Code で開いていること。`.github/agents/` 配下のエージェントファイルが自動的に認識されます。

#### 手順

1. **Copilot Chat を開く**
   - `Ctrl + Alt + I`（Windows）でチャットパネルを開きます

2. **エージェントを選択する**
   - チャット入力欄で `@Feature Builder` とタイプするか、エージェントドロップダウンから **Feature Builder** を選択します

3. **機能リクエストを送信する**
   - 以下のようなプロンプトを入力します

#### プロンプト例

**例 1: REST API エンドポイントの追加**
```
ユーザー一覧を返す GET /api/users エンドポイントを Express.js で実装してください。
ページネーション対応で、レスポンスは JSON 形式にしてください。
```

**例 2: ユーティリティ関数の作成**
```
日付を「YYYY年MM月DD日」形式にフォーマットする TypeScript ユーティリティ関数を作成してください。
タイムゾーンは Asia/Tokyo を考慮してください。
```

**例 3: 既存機能の拡張**
```
現在の認証モジュールに、パスワードリセット機能を追加してください。
メール送信によるトークンベースのリセットフローにしてください。
```

#### 実行フロー

エージェントに機能リクエストを送信すると、以下の順序で自動的に処理が進みます：

```
あなた ──リクエスト──▶ Feature Builder (コーディネーター)
                            │
                            ├──▶ Planner          タスクに分解
                            │       │
                            │       ▼
                            ├──▶ Plan Architect    計画を検証 ──フィードバック──▶ Planner
                            │
                            ├──▶ Implementer       コードを実装
                            │       │
                            │       ▼
                            └──▶ Reviewer          レビュー ──修正依頼──▶ Implementer
                                                               │
                                                               ▼
                                                          最終結果を返却
```

各サブエージェントの結果はチャット内で折りたたみ可能なブロックとして表示されます。展開すると、各ワーカーがどのツールを呼び出し、何を生成したかを確認できます。

#### うまくいかないとき

| 症状 | 対処法 |
|---|---|
| `@Feature Builder` が候補に出ない | `.github/agents/Feature Builder.agent.md` がワークスペース内にあるか確認 |
| サブエージェントが動作しない | VS Code 設定で `runSubagent` ツールが有効か確認（`chat.agent.tools` を検索） |
| 計画だけで実装されない | プロンプトに「実装まで完了してください」と明記する |

</details>

### 2. Multi-perspective Code Review パターン

複数の観点から並行してコードレビューを実施し、結果を統合します。

| エージェント | 役割 |
|---|---|
| **Thorough Reviewer** | 4つの観点（正確性・品質・セキュリティ・アーキテクチャ）で並行レビュー |

**使い方:** チャットで `@Thorough Reviewer` を選択し、レビュー対象のコードやファイルを指定します。

### 3. TDD パターン

テスト駆動開発のRed-Green-Refactorサイクルをサブエージェントで実行します。

| エージェント | 役割 | ユーザー呼び出し |
|---|---|---|
| **TDD** | コーディネーター。TDDサイクルを管理 | ✅ |
| **Red** | 失敗するテストを書く | ❌ (サブエージェント専用) |
| **Green** | テストを通す最小限のコードを書く | ❌ (サブエージェント専用) |
| **Refactor** | コード品質を改善 | ❌ (サブエージェント専用) |

**使い方:** チャットで `@TDD` を選択し、実装したい機能を記述します。

### 4. Recursive パターン

分割統治法でリストを再帰的に処理します。サブエージェントが自分自身を呼び出すパターンです。

| エージェント | 役割 |
|---|---|
| **RecursiveProcessor** | 4件超のリストは分割して自身に委譲、4件以下は直接処理 |

**使い方:** チャットで `@RecursiveProcessor` を選択し、処理したいリストを渡します。

## 前提条件

- VS Code で GitHub Copilot が有効であること
- `runSubagent` ツールが有効であること
- **Recursive パターン**を使用する場合: `chat.subagents.allowInvocationsFromSubagents` 設定を `true` にすること

## ファイル構成

```
.github/agents/
├── Feature Builder.agent.md   # Coordinator & Worker コーディネーター
├── Planner.agent.md           # ワーカー: 計画
├── Plan Architect.agent.md    # ワーカー: 設計検証
├── Implementer.agent.md       # ワーカー: 実装
├── Reviewer.agent.md          # ワーカー: レビュー
├── Thorough Reviewer.agent.md # Multi-perspective レビュー
├── TDD.agent.md               # TDD コーディネーター
├── Red.agent.md               # TDD: テスト作成
├── Green.agent.md             # TDD: 実装
├── Refactor.agent.md          # TDD: リファクタリング
└── RecursiveProcessor.agent.md # 再帰処理
```
