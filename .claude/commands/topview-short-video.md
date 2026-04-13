# Topview ショート動画自動生成パイプライン

ユーザーが渡したテーマから、Pixar風3Dキャラクターが日本語で解説する30秒ショート動画を半自動で生成するスキル。

## 起動方法

ユーザーが `/topview-short-video` と入力するか、テーマを渡して呼び出す。

## 入力仕様

ユーザーから以下の形式でテーマを受け取る。未指定の項目にはデフォルト値を使用する。

```
【テーマ】[トピック名]（必須）
【3選の観点】[どんな切り口で3選を選ぶか]（デフォルト: 便利な機能や特徴）
【ターゲット視聴者】[想定視聴者層]（デフォルト: 一般のテック好き層）
【トーン】[親しみやすい/専門的/カジュアル 等]（デフォルト: 親しみやすい）
【その他要望】[任意]
```

テーマが未入力の場合は、AskUserQuestion で入力を促す。

---

## 処理フロー

```
[入力受付] → [Step 1] 情報収集 → [Step 2] 3選選定 → [Step 3] 台本生成
  → 【🛑 停止1】台本承認 → [Step 4] キャラ画像生成
  → 【🛑 停止2】画像承認 → [Step 5] 動画生成 → [Step 6] 完成出力
```

---

## Step 0: セッション準備

1. タイムスタンプ付きの出力ディレクトリを作成する:
   ```
   ./topview_output/session_YYYYMMDD_HHMMSS/
   ```
2. TodoWrite でパイプライン全体の進捗を管理する

---

## Step 1: 情報収集

- WebSearch でテーマに関する最新情報を収集する
- 3選の候補となる機能・特徴・Tips を **5〜7個** リストアップする
- 各候補に「なぜ視聴者に刺さるか」の根拠をメモする
- 信頼性の高いソース（公式サイト、公式ドキュメント等）を優先
- 結果を `01_research_summary.md` に保存し、ターミナルにもサマリー表示

---

## Step 2: 3選の選定と構成案作成

以下の基準で3つを選定する:
- ターゲット視聴者に最も刺さる
- 30秒の短尺で伝えやすい
- 3つ並べた時にバリエーションがある

ユーザー要望に特別指定（メタ要素等）がある場合は該当する選に組み込む。
構成案を `02_structure.md` に保存し、ターミナルに表示。

---

## Step 3: 10秒×3のセリフ台本生成

### 台本ルール
- 各シーン: 10秒程度で読み切れる日本語セリフ（70〜90文字）
- トーンはユーザー指定に従う（デフォルト: 親しみやすい女性キャラの一人語り）
- 視聴者（カメラ）に語りかける形式

### 出力フォーマット

```
━━━━━━━━━━━━━━━━━━━━━━
🎬 シーン1（0:00-0:10）【選1のタイトル】
━━━━━━━━━━━━━━━━━━━━━━
セリフ：「...(70〜90文字)」
（文字数：XX文字）

━━━━━━━━━━━━━━━━━━━━━━
🎬 シーン2（0:10-0:20）【選2のタイトル】
━━━━━━━━━━━━━━━━━━━━━━
セリフ：「...(70〜90文字)」
（文字数：XX文字）

━━━━━━━━━━━━━━━━━━━━━━
🎬 シーン3（0:20-0:30）【選3のタイトル】
━━━━━━━━━━━━━━━━━━━━━━
セリフ：「...(70〜90文字)」
（文字数：XX文字）

📊 合計：XXX文字（目安：210〜270文字）
```

台本を `03_script_final.md` に保存。

---

## 🛑 停止ポイント1: 台本確定

**必ずユーザーの明示的な承認を待つ。自動で次に進んではならない。**

AskUserQuestion で以下の選択肢を提示:
- **OK、これで進めて** → Step 4 へ
- **修正したい** → 修正指示を受けて Step 3 に戻る（ループ）
- **中止** → パイプライン中断

修正指示を受けた場合は最新版のみを返す（過去バージョンを混在させない）。
修正ループは何回でも許容。

---

## Step 4: キャラクター画像生成（Nano Banana 2）

### 使用ツール
Renoise CLI (`video-maker:renoise-gen` スキル) の nano-banana-2 モデル

### 手順

1. Renoise のクレジット残高を確認:
   ```bash
   node {renoise-cli} credit me
   ```

2. 以下のプロンプトで 9:16 縦型画像を生成:
   ```bash
   node {renoise-cli} task generate \
     --prompt "<キャラクタープロンプト>" \
     --model nano-banana-2 --resolution 2k --ratio 9:16
   ```

3. 生成された画像をダウンロードして `04_character_image.png` に保存

### デフォルトキャラクタープロンプト

```
A Pixar-style 3D animated character for a vertical short video (9:16
aspect ratio).

STYLE: Pixar 3D animation aesthetic, similar to Monsters Inc. and
Inside Out. Smooth polished surfaces, colorful, friendly yet slightly
cool. Not overly cute, with a hint of coolness.

CHARACTER: A single cute female character, looks like early 20s,
approachable and intelligent, engineer-type girl.

APPEARANCE:
- Short bob or medium-length dark brown hair
- Wearing large round-frame or black-rimmed glasses
- Big expressive eyes with the lively Pixar-character look
- Casual engineer-style outfit: graphic T-shirt under a relaxed
  hoodie or cardigan, denim or chino pants
- Overall muted palette (white, gray, navy, beige) with one pop
  accent color (e.g., yellow sneakers)
- Energetic, bright, talkative personality expression

BACKGROUND: Clean modern creator's workspace. Large desk, dual
monitors, keyboard, indoor plants, stylish posters and lights on
the wall. A bit futuristic and tech-vibe workspace. Warm lighting
with cozy atmosphere.

COMPOSITION: Medium shot from chest up, character facing camera,
ready to speak. Vertical 9:16 format. Character positioned in
upper-center of frame to allow space for subtitles at bottom.

QUALITY: Ultra high quality, Pixar film production level, 4K,
detailed lighting, character clearly visible and in sharp focus.
```

ユーザーがキャラクターの変更を希望する場合はプロンプトを調整する。

### Renoise CLI パス

video-maker:renoise-gen スキルを呼び出して使用する。CLIパスはスキル内で自動解決される。

---

## 🛑 停止ポイント2: 画像確定

**必ずユーザーの明示的な承認を待つ。**

AskUserQuestion で以下の選択肢を提示:
- **OK、これで進めて** → Step 5 へ
- **同じプロンプトで再生成** → 同じプロンプトで Step 4 を再実行
- **プロンプトを微調整したい** → 修正指示を反映して Step 4 を再実行
- **中止** → パイプライン中断

---

## Step 5: 日本語音声+テロップ付き動画生成（Topview Avatar4）

### 使用ツール
Topview AI の Avatar4 モジュール (`topview-skill`)

### 手順

1. 依存パッケージを確認:
   ```bash
   pip3 install --break-system-packages -r {topview-skill-base}/scripts/requirements.txt
   ```

2. Topview AI のクレジット残高を確認:
   ```bash
   python3 {topview-skill-base}/scripts/user.py credit
   ```

3. 日本語女性ボイスを選択（デフォルト: Yuka）:
   ```bash
   python3 {topview-skill-base}/scripts/voice.py list --language ja --gender female
   ```

4. キャプションスタイルを取得:
   ```bash
   python3 {topview-skill-base}/scripts/avatar4.py list-captions
   ```

5. デフォルトボードIDを取得:
   ```bash
   python3 {topview-skill-base}/scripts/board.py list --default -q
   ```

6. 3シーンのセリフを `<break time="1s"/>` で区切って1本のテキストに結合し、Avatar4 で動画生成:
   ```bash
   python3 {topview-skill-base}/scripts/avatar4.py run \
     --image {04_character_image.png のパス} \
     --text '{シーン1セリフ}<break time="1s"/>{シーン2セリフ}<break time="1s"/>{シーン3セリフ}' \
     --voice {voiceId} \
     --caption {captionId} \
     --board-id {boardId} \
     --output {05_video_final.mp4 のパス} \
     --timeout 1200
   ```

7. タイムアウトした場合は `query` で再ポーリング:
   ```bash
   python3 {topview-skill-base}/scripts/avatar4.py query \
     --task-id {taskId} --timeout 600
   ```

### デフォルト設定

| 項目 | デフォルト値 |
|---|---|
| ボイス | Yuka (`Elu2nLYQkwhjIAtEOYlX8a2MSRaVOBgk`) |
| 言語 | 日本語 (ja) |
| キャプション | `1e7c01421c2a4597902bd6c8c249d26c` |
| モード | avatar4（高品質） |
| タイムアウト | 1200秒 |

### Topview Skill ベースパス

topview-skill スキル配下の `scripts/` ディレクトリを使用する。パスはプロジェクトルートからの相対パス `.claude/skills/topview-skill/scripts/` で解決する。

---

## Step 6: 完成動画の出力と提示

1. 動画ファイルを `05_video_final.mp4` としてセッションディレクトリに保存
2. ffprobe で動画メタデータ（尺、解像度、サイズ等）を取得
3. セッションログ `session_log.md` を作成（全工程の記録）
4. ターミナルに以下のサマリーを表示:

```
## 完成動画

ファイルパス: ./topview_output/session_XXXXXX/05_video_final.mp4

| 項目 | 内容 |
|---|---|
| テーマ | ... |
| 3選 | 1. ... / 2. ... / 3. ... |
| 総尺 | XX秒 |
| 解像度 | WxH（9:16 縦型） |
| 音声 | 日本語女性ボイス |
| テロップ | あり |
| ファイルサイズ | XX MB |

Topview プロジェクトリンク:
https://www.topview.ai/board/{boardId}?boardResultId={boardTaskId}
```

---

## 成果物構成

```
./topview_output/session_YYYYMMDD_HHMMSS/
  ├─ 01_research_summary.md
  ├─ 02_structure.md
  ├─ 03_script_final.md
  ├─ 04_character_image.png
  ├─ 05_video_final.mp4
  └─ session_log.md
```

---

## エラーハンドリング

| ステップ | エラー | 対応 |
|---|---|---|
| Step 1 | 情報収集失敗 | ユーザーに手動情報提供を促す |
| Step 3 | セリフ文字数超過 | 自動で短縮を試み、結果を提示 |
| Step 4 | 画像生成失敗 | 3回まで自動リトライ |
| Step 5 | タイムアウト | query で再ポーリング（最大2回） |
| Step 5 | タスク失敗 | 2回まで自動リトライ |
| 任意 | ユーザーが「中止」 | 現在の成果物を保存してパイプライン終了 |

---

## 制約事項

1. 既存の特定アニメ・映画キャラクターを模倣しない
2. 実在人物の顔を参照しない
3. 停止ポイント1・2で必ず停止し、承認なしに次のステップに進まない
4. 各ステップで何をしているかターミナルに逐次表示する
5. API 呼び出し回数を必要最小限にする
