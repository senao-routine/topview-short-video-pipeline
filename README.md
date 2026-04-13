# Topview AI x Claude Code 全自動ショート動画生成パイプライン

テーマを入力するだけで、**Pixar風3Dキャラクターが日本語で解説する30秒ショート動画**を、ブラウザを一切開かずにClaude Codeのターミナル上だけで自動生成するパイプラインです。

## デモ

> テーマ「Claude Code - 開発者に役立つ便利機能3選」で生成した例

```
入力: テーマを渡すだけ
  ↓ 情報収集 → 3選選定 → 台本生成 → 🛑承認 → キャラ画像生成 → 🛑承認 → 動画生成
出力: 日本語音声・テロップ付きの30秒ショート動画
```

## 何ができる?

| ステップ | 内容 | 使用ツール |
|----------|------|------------|
| 情報収集 | テーマに関する最新情報をWeb検索で自動収集 | Claude Code |
| 構成案作成 | 「〇〇できること3選」形式で構成を自動設計 | Claude Code |
| 台本生成 | 10秒x3シーン = 30秒のセリフ台本を自動作成 | Claude Code |
| キャラ画像 | Pixar風3Dキャラクターを AI で自動生成 | Nano Banana 2 (Renoise) |
| 動画生成 | 日本語音声 + リップシンク + テロップ付き動画を自動生成 | Topview AI Avatar4 |

## クイックスタート

### 前提条件

- [Claude Code](https://claude.com/product/claude-code) がインストール・認証済み
- [Topview AI](https://www.topview.ai) アカウントが認証済み
- Topview Skill がインストール済み
- Renoise (video-maker) Skill がインストール済み

### セットアップ

```bash
# リポジトリをクローン
git clone https://github.com/senao-routine/topview-short-video-pipeline.git
cd topview-short-video-pipeline

# Topview Skill の依存パッケージをインストール
pip install -r .agents/skills/topview-skill/scripts/requirements.txt
```

### 使い方

Claude Code を起動して、以下のいずれかの方法で実行します。

**方法1: スラッシュコマンド**

```
/topview-short-video
```

**方法2: 要件定義書を読み込ませる**

```
このファイルを読んで、指示に従ってタスクを実行してください: topview_pipeline_spec.md
```

**方法3: テーマを直接渡す**

```
【テーマ】Claude Code
【3選の観点】開発者に役立つ便利機能
【ターゲット視聴者】AI・プログラミングに関心のある開発者
【トーン】親しみやすく、でも技術的な正確さは保つ
【その他要望】最後の1選にメタ要素を入れたい
```

### 処理フロー

```
[テーマ入力]
   ↓
[Step 1] Web検索で情報収集
   ↓
[Step 2] 3選の選定と構成案作成
   ↓
[Step 3] 10秒x3のセリフ台本生成
   ↓
 🛑 停止ポイント1 → ユーザーが台本を確認・承認
   ↓
[Step 4] Pixar風キャラクター画像を生成 (Nano Banana 2)
   ↓
 🛑 停止ポイント2 → ユーザーが画像を確認・承認
   ↓
[Step 5] 日本語音声 + テロップ付き動画を生成 (Topview Avatar4)
   ↓
[Step 6] 完成動画を出力
```

2箇所の承認ポイントがあり、台本やキャラクター画像は何度でも修正・再生成できます。

## 出力ファイル

```
topview_output/session_YYYYMMDD_HHMMSS/
  ├── 01_research_summary.md   # 情報収集結果
  ├── 02_structure.md          # 3選の構成案
  ├── 03_script_final.md       # 確定した台本
  ├── 04_character_image.png   # Pixar風キャラクター画像
  ├── 05_video_final.mp4       # 完成動画（日本語音声+テロップ付き）
  └── session_log.md           # 全工程のログ
```

## プロジェクト構成

```
.
├── topview_pipeline_spec.md           # 要件定義書（パイプライン全体の設計）
├── .claude/
│   ├── skills/
│   │   ├── topview-short-video.md     # パイプラインスキル定義
│   │   └── topview-skill -> ...       # Topview AI公式スキル
│   └── commands/
│       └── topview-short-video.md     # /topview-short-video コマンド
├── .agents/skills/topview-skill/      # Topview AI公式スキル本体
│   ├── scripts/                       # Python実行スクリプト群
│   └── references/                    # APIリファレンスドキュメント
└── topview_output/                    # 生成物の出力先
```

## 動画仕様

| 項目 | 値 |
|------|-----|
| フォーマット | MP4 (H.264 + AAC) |
| アスペクト比 | 9:16（縦型） |
| 目標尺 | 約30秒 |
| 音声 | 日本語女性ボイス（TTS） |
| テロップ | 自動生成・表示 |
| キャラクター | Pixar風3Dアニメーション |

## コスト目安

1回のパイプライン実行あたり:

| 項目 | コスト |
|------|--------|
| キャラクター画像 (Nano Banana 2) | 12 Renoise credits |
| 動画生成 (Topview Avatar4) | 数 Topview credits |

## 注意事項

- Topview AI と Renoise のアカウント・クレジットが必要です
- APIキーは環境変数で管理されるため、リポジトリには含まれていません
- 生成された動画・画像ファイルは `.gitignore` で除外されています

## ライセンス

このプロジェクトの要件定義書・スキル定義は自由に利用できます。
Topview AI公式スキル（`.agents/skills/topview-skill/`）のライセンスは [LICENSE.txt](.agents/skills/topview-skill/LICENSE.txt) を参照してください。
