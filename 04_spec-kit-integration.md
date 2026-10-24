# 04. 仕様駆動の重量選択（Spec Kit の使い分け）

> **このファイルの責務:** 仕様駆動を「どの重量で回すか」の判断。3層モデル（What / How / 制約）、使い分けトリガー、そして重量級ツール Spec Kit を**主運用にしなかった判断**の記録（動かして評価した記録ではない——出自は §3）。テンプレート・構成例は `05_templates-and-patterns.md` を参照。

## 前提思想

> **仕様駆動は「やるか否か」ではなく「どの重量で回すか」の問題。What（仕様）はツール非依存で固定でき、重量級の Spec Kit は中〜大規模・多論点の変更でのみ割に合う。**
>
> 実践では、多くのタスクは `仕様Markdown + Agent Skills` の軽量運用で足り、Spec Kit のフルフローを要する場面は無かった。本章はその判断の記録であり、Spec Kit を「使うと決めたときの入口」も最小限だけ残す。
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

## 3. Spec Kit を主運用にしなかった判断

Spec Kit は GitHub 公式の仕様駆動開発（SDD: Spec-Driven Development）ツールキット（MIT License）。仕様→計画→タスク→実装のワークフローを、エージェントから呼ぶコマンド（Claude Code では Skill）で提供する。**現状は主運用に採用していない。**

> **出自 `[Assumption]`（著者の記憶。当時の記録は残っていない）:** 本節は、Spec Kit を動かして評価した記録ではない。**2025-12 頃の情報を前提に、2026-03 の本ガイドのごく初期のプロトタイプで、動かさずに当時のモデル（Opus 4.8 前後）の学習知識を元に定義した**判断である。2026-10-23 に別系統の生成AI のクロスレビュー（`01` §2 の段階3）で §4 の陳腐化が見つかり、Spec Kit `v1.2.3` で `specify init` だけを実測した。**フルフローは今も動かしていない。**

**採用しなかった理由**（いずれも動かして確かめた物ではない）:
- **対象タスクの規模が合わなかった `[Assumption]`:** 要件が明確で小〜中規模の変更は `仕様Markdown + Skills` で完結し、Spec Kit フルフローは不要だった（`02_mcp-and-tools-ecosystem.md` セクション1.3 の適用例）。
- **学習・初期コスト `[Assumption]`:** Constitution 設計やフェーズ運用に初期コストがかかり、軽量タスクでは投資対効果が出にくい。
- **Token効率 `[Assumption]`:** 機能毎の仕様ファイル（`specs/<NNN-name>/` の spec・plan・tasks）は、実装フェーズで大量に消費しうる。加えて、CLI のプロセスは常駐しないが、Claude Code 向けには `/speckit-*` の Skill が入り、既定ではその説明文が Skill 一覧の予算を消費する `[Fact]`（公式の [Claude Code の連携の説明](https://github.github.io/spec-kit/integrations/claude.html)。初期化の時に `--integration-options="--no-model-invocation"` を付けると、中核のコマンドの Skill は人間が呼ぶ時だけ動き、説明文が文脈から外れる）。

**Spec Kit が割に合う条件（採用を再検討すべき場面）:**
- 論点が多い／中〜大規模の機能追加
- 複数の技術スタックで並行実装を試す（Creative Exploration）
- 既存コードベースで破壊的変更のガードレール（Constitution）を明示的に効かせたい

> **正直な但し書き:** Spec Kit のフルフローは一度も動かしていない（`init` だけを 2026-10-23 に実測した）。上記は「動かさずに定義した現時点の判断」であり、大型タスクが来たら検証して更新する。

<!-- BRAINSTORM: 中〜大規模タスクが来たら Spec Kit フルフローを実際に試し、Token消費・使い勝手を検証したい -->

---

## 4. 使うと決めたときの入口（最小リファレンス）

Spec Kit を採用する場合の最小手順のみ示す。詳細は公式ドキュメントに委ねる。

> **確認日 2026-10-23・Spec Kit `v1.2.3`**（公式の Installation Guide・Quick Start と照らし、`specify init` を実測した）。Spec Kit は版が速く進み、本節の旧版の手順もそれで動かなくなった。⇒ **使う前に、公式の [Installation Guide](https://github.github.io/spec-kit/installation.html) と照らすこと。**

**インストール（版を固定した常設のインストール。公式の推奨経路）と初期化（Claude Code を指定）:**

```bash
# vX.Y.Z は Releases のタグ（先頭の v を残す）
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z

# 既存のプロジェクトに入れる（Linux / macOS は --script sh、Windows の PowerShell は --script ps）
specify init --here --force --integration claude --script sh
```

- **空でないディレクトリには `--force` が要る**（合流の確認）。公式は、先に既存の変更を commit するかバックアップを取ることを求めている。
- **初期化すると、`.specify/`（memory・scripts・templates 等）と、Claude Code 用の Skill（`.claude/skills/speckit-<command>/`）が追加される。** 既存の `CLAUDE.md`・`.gitignore` は書き換えなかった `[Fact]`（2026-10-23 の実測。他の既存ファイルは確かめていない）。機能毎の成果物は、リポジトリ直下の `specs/<NNN-name>/`（spec.md・plan.md・tasks.md）に置かれる。
- **Claude Code では `/speckit-specify` のようにハイフンで呼ぶ。** 公式の references はドット形（`/speckit.specify`）で書かれている（公式の [Command Invocation](https://github.github.io/spec-kit/integrations/cli.html#command-invocation)）。下の表は Claude Code の呼び名で書く。

**コマンドと2つの流れ:** 公式の [Quick Start](https://github.github.io/spec-kit/quickstart.html) は、小さめの機能向けの「短い流れ」と、本番の機能向けの「全部の流れ」を分けている——**§2 の重量の選び分けと同じ軸が、Spec Kit の中にもある。**

| コマンド | 役割 | 短い流れ | 全部の流れ |
|---------|------|:---:|:---:|
| `/speckit-constitution` | プロジェクトの原則・制約を定義（プロジェクトに1回） | ○ | ○ |
| `/speckit-specify` | 要件を定義（What/Why。技術スタックは書かない） | ○ | ○ |
| `/speckit-clarify` | 曖昧な所を質問で潰し、答えを spec に戻す | — | ○ |
| `/speckit-plan` | 技術スタック・アーキテクチャの選択 | ○ | ○ |
| `/speckit-checklist` | 要件の品質のチェックリストを作る | — | ○ |
| `/speckit-tasks` | 実装可能な粒度にタスク分解 | ○ | ○ |
| `/speckit-analyze` | spec・plan・tasks の間の矛盾・抜けを報告する（読むだけ） | — | ○ |
| `/speckit-implement` | タスクを実行（ここで Agent Skills が手順を提供） | ○ | ○ |
| `/speckit-converge` | コードを成果物と突き合わせ、残りを tasks に足す | ○ | ○ |

**Token消費を抑える運用:**
1. 1機能 = 1 Gitブランチ = 1 セッション で進める——**これは整理の仕方であって、Spec Kit の仕組みではない。** Spec Kit は `.specify/feature.json` が指す機能のディレクトリで今の機能を決め、チェックアウト中のブランチは見ない（Git 自体が要らない）。番号付きのブランチは、opt-in の git 拡張が担う
2. 仕様が大きい場合はフェーズを分割して `/speckit-implement` を複数回実行
3. 完了した機能の `specs/<NNN-name>/` は、次のセッションで読ませない

---

## 5. Constitution と CLAUDE.md の重複回避（採用時のみ）

Spec Kit を使う場合、Constitution（Spec Kit の原則ファイル）と CLAUDE.md（常時ロードの制約）で同じ内容を二重に書かない。一方が「正本」、他方は「参照」とする。

> **効く場面で分ける:** Constitution は **Spec Kit の流れの中**の正本（以後の各手順は、これに照らして評価される——公式 Quick Start）、`CLAUDE.md` は**全ての作業で常に効く**正本。Spec Kit の流れの外で実装するときに効くのは `CLAUDE.md` である。

| 内容 | 正本 | もう片方の書き方 |
|------|------|----------------|
| 技術スタック | `CLAUDE.md` | Constitution では「`CLAUDE.md` を参照」と記載 |
| コーディング規約 | `CLAUDE.md` | Constitution では「`CLAUDE.md` の規約に従う」と記載 |
| アーキテクチャ原則 | Constitution | `CLAUDE.md` には「仕様定義時はConstitutionの原則に従う」と一言のみ |
| 破壊的変更ポリシー | Constitution | `CLAUDE.md` への記載は不要（仕様定義時のみ関係） |
