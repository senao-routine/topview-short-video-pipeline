# Topview AI x Claude Code 全自動ショート動画生成パイプライン

**テーマ入力** または **本編動画ファイル** を渡すだけで、Pixar風3Dキャラクターが日本語で解説する30秒ショート動画を、ブラウザを一切開かずにClaude Codeのターミナル上だけで自動生成するパイプラインです。

## 2つの入力モード

### モードA: テーマ入力

テキストでテーマを渡すと、Web検索で情報を収集し、3選を自動選定してショート動画を生成します。

```
入力: テーマ（例:「Claude Code × 便利機能3選」）
  ↓ Web検索 → 3選選定 → 台本生成 → 🛑承認 → キャラ画像×3 → 🛑承認 → 動画生成
出力: 日本語音声・テロップ付きの30秒ショート動画
```

### モードB: 本編動画入力（推奨）

本編動画ファイルを渡すと、AIが内容を分析・文字起こしし、視聴者の目を引く3トピックを自動抽出してティーザー型ショート動画を生成します。

```
入力: 本編動画ファイル（input_videos/ に配置）
  ↓ AI分析 → 3トピック抽出 → 台本生成 → 🛑承認 → キャラ画像×3 → 🛑承認 → 動画生成
出力: 本編への誘導を意識した30秒ショート動画
```

## 特徴

| 機能 | 説明 |
|------|------|
| シーン切り替え | 3シーンごとに背景・シチュエーションが自動で切り替わる |
| キャラクター自動設計 | テーマに最適なキャラクターの外見・服装を自動生成 |
| ボイス自動選択 | キャラクターの性別に応じて日本語ボイスを自動切替（男性3種ランダム / 女性1種） |
| テロップ | シンプルな白文字テロップを自動表示 |
| 動画分析 | 本編動画をAIが文字起こし・要約し、ハイライトを自動抽出（モードB） |

## 何ができる?

| ステップ | 内容 | 使用ツール |
|----------|------|------------|
| 動画分析 | 本編動画の文字起こし・要約・トピック抽出（モードB） | Gemini 3.1 Pro |
| 情報収集 | テーマに関する最新情報をWeb検索で自動収集（モードA） | Claude Code |
| 構成案作成 | 3トピック + シーンごとのシチュエーションを自動設計 | Claude Code |
| 台本生成 | 10秒x3シーン = 30秒のセリフ台本を自動作成 | Claude Code |
| キャラ画像 | Pixar風3Dキャラクター × 3枚（背景違い）を生成 | Nano Banana 2 (Renoise) |
| 動画生成 | 日本語音声 + リップシンク + テロップ付き動画を生成 | Topview AI Avatar4 |
| 動画結合 | 3シーンを1本の動画に結合 | ffmpeg |

## クイックスタート

### 前提条件

- [Claude Code](https://claude.com/product/claude-code) がインストール・認証済み
- [Topview AI](https://www.topview.ai) アカウントが認証済み
- Topview Skill がインストール済み
- Renoise (video-maker) Skill がインストール済み
- ffmpeg がインストール済み

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

**方法1: スラッシュコマンド（推奨）**

```
/topview-short-video
```

モードA / モードB の選択を対話的に案内します。

**方法2: 本編動画を渡す（モードB）**

`input_videos/` フォルダに本編動画を配置してからスラッシュコマンドを実行します。

**方法3: テーマを直接渡す（モードA）**

```
【テーマ】Claude Code
【3選の観点】開発者に役立つ便利機能
【ターゲット視聴者】AI・プログラミングに関心のある開発者
【トーン】親しみやすく、でも技術的な正確さは保つ
【その他要望】最後の1選にメタ要素を入れたい
```

### 処理フロー

```
[入力受付] モードA or B を選択
   ↓
[Step 1] A: Web検索で情報収集 / B: 動画をAI分析・文字起こし
   ↓
[Step 2] 3トピック選定 + シーンごとのシチュエーション設計
   ↓
[Step 3] 10秒x3のセリフ台本生成
   ↓
 🛑 停止ポイント1 → ユーザーが台本を確認・承認
   ↓
[Step 4] Pixar風キャラクター画像 × 3枚を生成（背景はシーンごとに変更）
   ↓
 🛑 停止ポイント2 → ユーザーが画像を確認・承認
   ↓
[Step 5] 日本語音声 + テロップ付き動画を3シーン個別に生成
   ↓
[Step 6] ffmpegで結合 → 完成動画を出力
```

2箇所の承認ポイントがあり、台本やキャラクター画像は何度でも修正・再生成できます。

## 出力ファイル

```
topview_output/session_YYYYMMDD_HHMMSS/
  ├── 01_research_summary.md     # テーマ情報収集（モードA）
  ├── 01_video_analysis.md       # 動画分析・文字起こし結果（モードB）
  ├── 02_structure.md            # 3トピック構成案
  ├── 03_script_final.md         # 確定した台本
  ├── 04_character_scene1.png    # シーン1のキャラクター画像
  ├── 04_character_scene2.png    # シーン2のキャラクター画像
  ├── 04_character_scene3.png    # シーン3のキャラクター画像
  ├── 05_scene1.mp4              # シーン1の動画
  ├── 05_scene2.mp4              # シーン2の動画
  ├── 05_scene3.mp4              # シーン3の動画
  ├── 05_video_final.mp4         # 完成動画
  └── session_log.md             # 全工程のログ
```

## プロジェクト構成

```
.
├── topview_pipeline_spec.md           # 要件定義書（パイプライン全体の設計）
├── input_videos/                      # 本編動画の配置先（モードB）
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
| 目標尺 | 約30秒（10秒 × 3シーン） |
| 音声 | 日本語ボイス（キャラクター性別に応じて自動選択） |
| テロップ | シンプル白文字（自動生成） |
| キャラクター | Pixar風3Dアニメーション（テーマに応じて自動設計） |
| シーン切替 | 3シーンで背景・シチュエーションが変化 |

## ボイス設定

キャラクターの性別に応じて自動で切り替わります。

| 性別 | ボイス | 備考 |
|------|--------|------|
| 女性 | Yuka | 固定 |
| 男性 | Haruto / Tomoya / Yuki outdoors | 3つからランダム選択 |

## コスト目安

1回のパイプライン実行あたり:

| 項目 | コスト |
|------|--------|
| キャラクター画像 × 3枚 (Nano Banana 2) | 36 Renoise credits |
| 動画生成 × 3シーン (Topview Avatar4) | 数 Topview credits |

## 注意事項

- Topview AI と Renoise のアカウント・クレジットが必要です
- APIキーは環境変数で管理されるため、リポジトリには含まれていません
- 生成された動画・画像ファイルは `.gitignore` で除外されています
- 入力動画ファイル（`input_videos/`）も `.gitignore` で除外されています

## ライセンス

このプロジェクトの要件定義書・スキル定義は自由に利用できます。
Topview AI公式スキル（`.agents/skills/topview-skill/`）のライセンスは [LICENSE.txt](.agents/skills/topview-skill/LICENSE.txt) を参照してください。
