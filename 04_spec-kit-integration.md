# 04. 仕様駆動の重量選択（Spec Kit の評価と使い分け）

> **このファイルの責務:** 仕様駆動を「どの重量で回すか」の判断。3層モデル（What / How / 制約）、使い分けトリガー、そして重量級ツール Spec Kit を評価した上で**主運用にしなかった判断**の記録。テンプレート・構成例は `05_templates-and-patterns.md` を参照。

## 前提思想

> **仕様駆動は「やるか否か」ではなく「どの重量で回すか」の問題。What（仕様）はツール非依存で固定でき、重量級の Spec Kit は中〜大規模・多論点の変更でのみ割に合う。**
>
> 実践では、多くのタスクは `仕様Markdown + Agent Skills` の軽量運用で足り、Spec Kit のフルフローは過剰だった。本章はその判断の記録であり、Spec Kit を「使うと決めたときの入口」も最小限だけ残す。
>
> **`[Judgment]` 軽量運用は「仕様を書かない」ことではない。** モデルが強力になるほど、やりたいことを雑に投げて丸ごと任せたくなる。しかし出力がもっともらしくなるほど**反証（falsify）は難しくなる**ため、What を検証可能な形で固定する規律の価値は、下がるどころか**上がる**。軽くしてよいのは実現手段（ツールの重量）であって、What と受入基準（AC）を固定する規律ではない。

---

## 1. 仕様駆動の役割分担

### 1.1 3層モデル（What / How / 制約）

仕様駆動を支える役割分担。**What層はツール非依存**で、重量に応じて実現手段を選ぶ。

```
What層（何を作るか）  → 仕様で固定する層
  ├─ 軽量: 仕様Markdown（要件が明確／論点が少ない）
  └─ 重量: Spec Kit（論点が多い／中〜大規模）
How層（どう作るか）   → Agent Skills（タスクの実行手順）
制約層（常に守る）    → CLAUDE.md（ユーザー／プロジェクトメモリ）
```

> **methodology と content の分離:** この3層は「再利用可能な方法論（How = Agent Skills）」と「プロジェクト固有のcontent（What = 仕様）」を役割分離する構造でもある。方法論はプロジェクト横断で再利用し、固有contentは仕様側に置く（詳細は `03_agent-skills-knowhow.md` セクション2）。

### 1.2 比較マトリクス

| 観点 | What層（仕様） | How層（Agent Skills） | 制約層（CLAUDE.md） |
|------|---------------|----------------------|--------------------|
| 目的 | 何を作るべきかを定義 | タスクの実行方法を教える | コーディング規約・振る舞いの制約 |
| 粒度 | 粗粒度（What / Why） | 細粒度（How） | 中粒度（制約） |
| ライフサイクル | 機能ごとに生成・発展 | 永続的（横断再利用可） | プロジェクト固有で永続 |
| Token消費 | 参照時に消費 | 関連時のみ読み込み | 常時コンテキストに含まれる |

---

## 2. 使い分けトリガー（重量の選択）

すべての変更で仕様駆動を重くする必要はない。**規模と論点数で重量を選ぶ。**

| 状況 | 推奨 | 理由 |
|------|------|------|
| バグ修正・小改修（1〜2ファイル） | CLAUDE.md + Skills のみ | 仕様フローがoverkill |
| 軽量変更（仕様を先に固定したい変更） | 仕様Markdown + Skills | Markdownに仕様を書いてAIに渡す。要件が明確なら十分 |
| 論点が複数ある変更 / 中〜大規模変更 | Spec Kit フルフロー | Specify → Plan → Tasks → Implement の価値が出る |
| 大規模リファクタリング | Spec Kit + Constitution見直し | アーキテクチャ変更をConstitutionで制御しつつ実施 |

> **補足:** 「仕様Markdown + Skills」は新機能に限定されない。簡易な仕様変更のように要件が明確・単純だが仕様を先に書きたい変更にも有効（適用例は `02_mcp-and-tools-ecosystem.md` セクション1.3）。

---

## 3. Spec Kit を主運用にしなかった判断（評価記録）

Spec Kit は GitHub 公式の仕様駆動開発（SDD: Spec-Driven Development）ツールキット（MIT License）。仕様→計画→タスク→実装のワークフローをスラッシュコマンドで提供する。評価の上で、**現状は主運用に採用していない。**

**却下した理由:**
- **対象タスクの規模が合わなかった:** 要件が明確で小〜中規模の変更は `仕様Markdown + Skills` で完結し、Spec Kit フルフローは不要だった（`02_mcp-and-tools-ecosystem.md` セクション1.3 の適用例）。
- **学習・初期コスト:** Constitution 設計やフェーズ運用に初期コストがかかり、軽量タスクでは投資対効果が出にくい。
- **Token効率:** Spec Kit 自体はCLIでベースライン増加はないが、`.specify/` の仕様ファイルは実装フェーズで大量消費しうる。

**Spec Kit が割に合う条件（採用を再検討すべき場面）:**
- 論点が多い／中〜大規模の機能追加
- 複数の技術スタックで並行実装を試す（Creative Exploration）
- 既存コードベースで破壊的変更のガードレール（Constitution）を明示的に効かせたい

> **正直な但し書き:** Spec Kit のフルフロー本格運用は未実証。上記は「評価と現時点の判断」であり、大型タスクが来たら検証して更新する。

<!-- BRAINSTORM: 中〜大規模タスクが来たら Spec Kit フルフローを実際に試し、Token消費・使い勝手を検証したい -->

---

## 4. 使うと決めたときの入口（最小リファレンス）

Spec Kit を採用する場合の最小手順のみ示す。詳細は公式ドキュメントに委ねる。

**初期化（Claude Code をエージェント指定）:**

```bash
# Linux / macOS
uvx --from git+https://github.com/github/spec-kit.git specify init . --ai claude --script sh

# Windows (PowerShell)
uvx --from git+https://github.com/github/spec-kit.git specify init . --ai claude --script ps
```

初期化後に `.specify/` が生成される（既存コードには影響しない）。以下のスラッシュコマンドが使えるようになる。

| コマンド | 役割 |
|---------|------|
| `/speckit.constitution` | プロジェクトの原則・制約を定義 |
| `/speckit.specify` | 要件を定義（What/Why。技術スタックは書かない） |
| `/speckit.plan` | 技術スタック・アーキテクチャの選択 |
| `/speckit.tasks` | 実装可能な粒度にタスク分解 |
| `/speckit.implement` | タスクを実行（ここで Agent Skills が手順を提供） |

**Token消費を抑える運用:**
1. 1機能 = 1 Gitブランチ = 1 セッション で進める（Spec KitはGitブランチで機能を検出）
2. 仕様が大きい場合はフェーズを分割して `/speckit.implement` を複数回実行
3. 完了した機能の `.specify/features/` は次のセッションで `@` 参照しない

---

## 5. Constitution と CLAUDE.md の重複回避（採用時のみ）

Spec Kit を使う場合、Constitution（Spec Kit の原則ファイル）と CLAUDE.md（常時ロードの制約）で同じ内容を二重に書かない。一方が「正本」、他方は「参照」とする。

| 内容 | 正本 | もう片方の書き方 |
|------|------|----------------|
| 技術スタック | `CLAUDE.md` | Constitution では「`CLAUDE.md` を参照」と記載 |
| コーディング規約 | `CLAUDE.md` | Constitution では「`CLAUDE.md` の規約に従う」と記載 |
| アーキテクチャ原則 | Constitution | `CLAUDE.md` には「仕様定義時はConstitutionの原則に従う」と一言のみ |
| 破壊的変更ポリシー | Constitution | `CLAUDE.md` への記載は不要（仕様定義時のみ関係） |
