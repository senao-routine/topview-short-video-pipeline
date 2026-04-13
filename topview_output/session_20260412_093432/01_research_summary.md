# Step 1: 情報収集結果サマリー

**テーマ:** Claude Code — 開発者に役立つ便利機能
**収集日:** 2026-04-12

---

## Claude Code 概要

- Anthropic が提供する AI コーディングエージェント
- ターミナル、VS Code、JetBrains、デスクトップアプリ、Webアプリ(claude.ai/code)で利用可能
- 2026年2月時点で GitHub パブリックコミットの4%（約13.5万件/日）が Claude Code による
- Pragmatic Engineer 調査で「最も愛されるツール」46%（2026年2月）

---

## 3選候補（5〜7個）

### 候補1: MCP サーバー連携
- Model Context Protocol で外部ツール（GitHub, DB, Slack, API等）と接続
- ターミナルから出ずに PR 作成、DB クエリ、デプロイが可能
- **刺さる理由:** 開発者のツール切り替えコストを劇的に削減

### 候補2: カスタムサブエージェント / エージェントチーム
- 複雑なタスクを並列で分割実行（split-and-merge パターン）
- 各エージェントが独立した git worktree で作業、コンフリクトなし
- **刺さる理由:** 大規模リファクタリングや並列タスクが現実的に

### 候補3: Skills（スキル）システム
- `.claude/skills/` にカスタムワークフローを定義 → `/スラッシュコマンド` で呼び出し
- 動画生成、デプロイ、コードレビュー等を1コマンドで自動化
- **刺さる理由:** 繰り返し作業を完全自動化、チーム共有も可能

### 候補4: Hooks（フック）
- ライフサイクルイベントに shell コマンドを自動実行
- リンティング、フォーマット、セキュリティチェックを確実に実行
- **刺さる理由:** プロンプトと違い「確実に実行される」保証がある

### 候補5: CLAUDE.md（プロジェクトルール）
- プロジェクト固有のルール・文脈を常時読み込み
- コーディング規約、アーキテクチャ方針を自然言語で記述
- **刺さる理由:** チーム全員が同じルールで AI を活用できる

### 候補6: 自律的コード実行（テスト→修正ループ）
- コードベース全体を読み、ファイル横断で変更、テスト実行
- テスト失敗 → エラー読解 → 修正 → 再実行を自動ループ
- **刺さる理由:** 開発者がレビューに集中できる

### 候補7: マルチプラットフォーム対応
- ターミナル CLI / VS Code / JetBrains / デスクトップアプリ / Web
- どこからでも同じ体験で使える
- **刺さる理由:** 環境を選ばない柔軟さ

---

## ソース

- [Claude Code 公式](https://claude.com/product/claude-code)
- [Claude Code Docs](https://code.claude.com/docs/en/overview)
- [Anthropic 公式](https://www.anthropic.com/product/claude-code)
- [Claude Code Review 2026 - Neuriflux](https://neuriflux.com/en/blog/claude-code-review-2026)
- [Claude Code Extensions Guide - Morph](https://www.morphllm.com/claude-code-extensions)
