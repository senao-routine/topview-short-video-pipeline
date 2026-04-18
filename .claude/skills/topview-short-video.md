# Topview ショート動画自動生成パイプライン

Pixar風3Dキャラクターが日本語で解説する30秒ショート動画を半自動で生成するスキル。
2つの入力モードに対応。

## 起動方法

ユーザーが `/topview-short-video` と入力するか、テーマまたは動画ファイルを渡して呼び出す。

---

## 入力モード

### モードA: テーマ入力モード

ユーザーがテキストでテーマを渡す。Web検索で情報を収集し、3選を自動選定する。

```
【テーマ】[トピック名]（必須）
【3選の観点】[どんな切り口で3選を選ぶか]（デフォルト: 便利な機能や特徴）
【ターゲット視聴者】[想定視聴者層]（デフォルト: 一般のテック好き層）
【トーン】[親しみやすい/専門的/カジュアル 等]（デフォルト: 親しみやすい）
【その他要望】[任意]
```

### モードB: 本編動画入力モード（推奨）

ユーザーが本編の動画ファイルを渡す。動画の内容を分析し、視聴者の目を引く3トピックを自動抽出してショート動画を生成する。

```
【動画ファイル】[input_videos/ 内のファイル名、またはフルパス]
【ターゲット視聴者】[想定視聴者層]（デフォルト: 本編の想定視聴者）
【トーン】[親しみやすい/専門的/カジュアル 等]（デフォルト: 親しみやすい）
【その他要望】[任意]
```

**モード判定:** ユーザーが動画ファイルを渡した場合はモードB、テーマテキストを渡した場合はモードA。
未指定の場合は AskUserQuestion でどちらのモードか確認する。

---

## 処理フロー

### モードA（テーマ入力）

```
[入力受付] → [Step 1A] Web検索で情報収集 → [Step 2] 3選選定 → [Step 3] 台本生成
  → 【🛑 停止1】台本承認 → [Step 4] キャラ画像生成（3シーン分）
  → 【🛑 停止2】画像承認 → [Step 5] 動画生成（3シーン個別） → [Step 6] 結合・完成出力
```

### モードB（動画入力）

```
[入力受付] → [Step 1B] 動画分析・文字起こし → [Step 2] 3トピック抽出 → [Step 3] 台本生成
  → 【🛑 停止1】台本承認 → [Step 4] キャラ画像生成（3シーン分）
  → 【🛑 停止2】画像承認 → [Step 5] 動画生成（3シーン個別） → [Step 6] 結合・完成出力
```

---

## Step 0: セッション準備

1. タイムスタンプ付きの出力ディレクトリを作成する:
   ```
   ./topview_output/session_YYYYMMDD_HHMMSS/
   ```
2. TodoWrite でパイプライン全体の進捗を管理する

---

## Step 1A: 情報収集（モードA: テーマ入力時）

- WebSearch でテーマに関する最新情報を収集する
- 3選の候補となる機能・特徴・Tips を **5〜7個** リストアップする
- 各候補に「なぜ視聴者に刺さるか」の根拠をメモする
- 信頼性の高いソース（公式サイト、公式ドキュメント等）を優先
- 結果を `01_research_summary.md` に保存し、ターミナルにもサマリー表示

---

## Step 1B: 動画分析・文字起こし（モードB: 動画入力時）

### 使用ツール
`video-maker:gemini-gen` スキル（Gemini 3.1 Pro による動画分析）

### 手順

1. ユーザーが渡した動画ファイルのパスを確認する
   - `input_videos/` 内のファイル名が渡された場合はフルパスに変換
   - フルパスが渡された場合はそのまま使用

2. `video-maker:gemini-gen` スキルで動画を分析する。以下の情報を抽出:
   - **文字起こし:** 動画内の音声を全文テキスト化
   - **内容要約:** 動画全体の要約（200〜300文字）
   - **主要トピック:** 動画内で語られている主要な話題を5〜7個リストアップ
   - **視聴者にとってのハイライト:** 最もインパクトのあるポイント
   - **動画の雰囲気・トーン:** 話者の語り口、テンション

3. 分析結果を `01_video_analysis.md` に保存し、ターミナルにサマリー表示

### 分析プロンプト例

```
この動画を詳細に分析してください:
1. 音声の完全な文字起こし（日本語）
2. 動画全体の要約（200〜300文字）
3. 主要トピックを5〜7個リストアップ（各トピックに1行の説明付き）
4. 視聴者が最も興味を持ちそうなハイライトポイント
5. 話者のトーンや雰囲気の特徴
```

---

## Step 2: 3トピックの選定と構成案作成

### モードA（テーマ入力時）
以下の基準で3つを選定する:
- ターゲット視聴者に最も刺さる
- 30秒の短尺で伝えやすい
- 3つ並べた時にバリエーションがある

### モードB（動画入力時）
Step 1B の分析結果から、**視聴者が最も目を引く3つのトピック**を抽出する:
- 本編を見たくなるような「おっ」と思わせるポイントを選ぶ
- 本編のネタバレになりすぎない程度に要約する
- 「続きは本編で」と思わせるティーザー的な内容にする
- 3つのトピックは異なる切り口にする（同じ話題の繰り返しにしない）

**各トピックにシチュエーション（背景・場面）を設定する:**

| シーン | トピック内容 | シチュエーション例 |
|--------|-------------|-------------------|
| シーン1 | トピック1の要約 | トピック1に最も合う場所・環境 |
| シーン2 | トピック2の要約 | トピック2に最も合う場所・環境 |
| シーン3 | トピック3の要約 | トピック3に最も合う場所・環境 |

構成案を `02_structure.md` に保存し、ターミナルに表示。

---

## Step 3: 10秒×3のセリフ台本生成

### 台本ルール
- 各シーン: 10秒程度で読み切れる日本語セリフ（70〜90文字）
- トーンはユーザー指定に従う（デフォルト: 親しみやすい女性キャラの一人語り）
- 視聴者（カメラ）に語りかける形式
- モードBの場合: 本編への誘導を意識した内容にする

### 出力フォーマット

```
━━━━━━━━━━━━━━━━━━━━━━
🎬 シーン1（0:00-0:10）【トピック1のタイトル】
🏠 シチュエーション: [この場面の背景・環境]
━━━━━━━━━━━━━━━━━━━━━━
セリフ：「...(70〜90文字)」
（文字数：XX文字）

━━━━━━━━━━━━━━━━━━━━━━
🎬 シーン2（0:10-0:20）【トピック2のタイトル】
🏠 シチュエーション: [この場面の背景・環境]
━━━━━━━━━━━━━━━━━━━━━━
セリフ：「...(70〜90文字)」
（文字数：XX文字）

━━━━━━━━━━━━━━━━━━━━━━
🎬 シーン3（0:20-0:30）【トピック3のタイトル】
🏠 シチュエーション: [この場面の背景・環境]
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

修正指示を受けた場合は最新版のみを返す。修正ループは何回でも許容。

---

## Step 4: キャラクター画像生成（Nano Banana 2）— 3シーン分

### 重要: シーンごとにシチュエーションを切り替える

キャラクター（CHARACTER + APPEARANCE）は **3シーン共通で固定**。
背景（BACKGROUND）は **シーンごとに変更** して、各トピックに合ったシチュエーションにする。

→ 合計 **3枚** の画像を生成する。

### 使用ツール
Renoise CLI (`video-maker:renoise-gen` スキル) の nano-banana-2 モデル

### 手順

1. Renoise のクレジット残高を確認（3枚分 = 約36クレジット）

2. テーマ/動画内容から、共通のキャラクター設定を決定する

3. 各シーンの背景を、Step 2 で設定したシチュエーションに基づいて決定する

4. 3枚の画像を生成:
   ```bash
   # シーン1
   node {renoise-cli} task generate \
     --prompt "<固定パート + シーン1のBACKGROUND>" \
     --model nano-banana-2 --resolution 2k --ratio 9:16

   # シーン2
   node {renoise-cli} task generate \
     --prompt "<固定パート + シーン2のBACKGROUND>" \
     --model nano-banana-2 --resolution 2k --ratio 9:16

   # シーン3
   node {renoise-cli} task generate \
     --prompt "<固定パート + シーン3のBACKGROUND>" \
     --model nano-banana-2 --resolution 2k --ratio 9:16
   ```

5. 生成された画像をダウンロード:
   - `04_character_scene1.png`
   - `04_character_scene2.png`
   - `04_character_scene3.png`

### キャラクタープロンプトの構成

#### 固定パート（毎回そのまま使用）

```
A Pixar-style 3D animated character for a vertical short video (9:16
aspect ratio).

STYLE: Pixar 3D animation aesthetic, similar to Monsters Inc. and
Inside Out. Smooth polished surfaces, colorful, friendly yet slightly
cool. Not overly cute, with a hint of coolness.

{CHARACTER}  ← 3シーン共通

{APPEARANCE}  ← 3シーン共通

{BACKGROUND}  ← シーンごとに変更

COMPOSITION: Medium shot from chest up, character facing camera,
ready to speak. Vertical 9:16 format. Character positioned in
upper-center of frame to allow space for subtitles at bottom.

QUALITY: Ultra high quality, Pixar film production level, 4K,
detailed lighting, character clearly visible and in sharp focus.
```

#### 可変パート: CHARACTER + APPEARANCE（3シーン共通）

テーマ・ターゲット視聴者・トーンから判断して英語で生成する。

**CHARACTER:**
- テーマの専門家・実践者として自然なキャラクターを設定する
- 年齢・性別・雰囲気はテーマに最も合うものを選ぶ
- 1キャラクターのみ（A single character）

**APPEARANCE:**
- テーマに合った服装・小物を具体的に指定する
- 髪型、目の特徴、表情も指定する
- カラーパレットはミュートトーン基調 + アクセントカラー1色
- Big expressive eyes with the lively Pixar-character look（固定）

#### 可変パート: BACKGROUND（シーンごとに変更）

各シーンのトピック・シチュエーションに合わせた背景を設定する。

**シーンごとの背景設計ルール:**
- トピックの内容を視覚的に表現する環境を選ぶ
- 3シーンで異なる場所にし、視覚的なバリエーションを出す
- Warm lighting with cozy atmosphere は共通で維持
- 小物・装飾でトピックの雰囲気を補強する

**例: テーマ「プログラミング学習のコツ3選」の場合**

| シーン | トピック | BACKGROUND |
|--------|---------|------------|
| 1 | エラーを楽しむ | Cozy home office with a laptop showing colorful error messages on screen, coffee mug with "Debug" text, warm desk lamp |
| 2 | 小さく作る | Minimalist workshop with building blocks, a small prototype on the desk, whiteboard with simple diagrams |
| 3 | コミュニティに参加 | Bright co-working space cafe, other people working in background (blurred), community event poster on wall |

#### 生成ルール

1. プロンプトは**必ず英語**で書くこと
2. CHARACTER と APPEARANCE は **3枚とも完全に同じ記述** にすること（キャラクター一貫性）
3. BACKGROUND のみシーンごとに変える
4. 各セクションは具体的・描写的に書く
5. ユーザーが `【その他要望】` でキャラクター指定をしている場合はそれを優先
6. 生成したプロンプト全文をターミナルに表示してから画像生成に進む

---

## 🛑 停止ポイント2: 画像確定

**3枚すべてをユーザーに提示し、明示的な承認を待つ。**

AskUserQuestion で以下の選択肢を提示:
- **OK、これで進めて** → Step 5 へ
- **特定のシーンを再生成** → 指定シーンのみ再生成
- **キャラクター自体を変えたい** → 3枚すべて再生成
- **中止** → パイプライン中断

---

## Step 5: 日本語音声+テロップ付き動画生成（Topview Avatar4）— 3シーン個別生成

### 重要: シーンごとに個別のアバター動画を生成し、最後に結合する

各シーンで異なるキャラクター画像（異なる背景）を使用するため、
**3つの個別アバター動画を生成**して ffmpeg で結合する。

### 使用ツール
- Topview AI の Avatar4 モジュール (`topview-skill`)
- ffmpeg（動画結合）

### 手順

1. 依存パッケージを確認:
   ```bash
   pip3 install --break-system-packages -r {topview-skill-base}/scripts/requirements.txt
   ```

2. Topview AI のクレジット残高を確認

3. デフォルトボードID・ボイス・キャプションを取得

4. **3シーンを個別に生成** — 並列で submit し、まとめて待機:
   ```bash
   # シーン1
   python3 {topview-skill-base}/scripts/avatar4.py submit \
     --image {04_character_scene1.png} \
     --text '{シーン1セリフ}' \
     --voice {voiceId} \
     --caption {captionId} \
     --board-id {boardId} -q

   # シーン2
   python3 {topview-skill-base}/scripts/avatar4.py submit \
     --image {04_character_scene2.png} \
     --text '{シーン2セリフ}' \
     --voice {voiceId} \
     --caption {captionId} \
     --board-id {boardId} -q

   # シーン3
   python3 {topview-skill-base}/scripts/avatar4.py submit \
     --image {04_character_scene3.png} \
     --text '{シーン3セリフ}' \
     --voice {voiceId} \
     --caption {captionId} \
     --board-id {boardId} -q
   ```

5. 各タスクの完了を待機:
   ```bash
   python3 {topview-skill-base}/scripts/avatar4.py query --task-id {taskId1} --timeout 600
   python3 {topview-skill-base}/scripts/avatar4.py query --task-id {taskId2} --timeout 600
   python3 {topview-skill-base}/scripts/avatar4.py query --task-id {taskId3} --timeout 600
   ```

6. 3本の動画をダウンロード:
   - `05_scene1.mp4`
   - `05_scene2.mp4`
   - `05_scene3.mp4`

7. ffmpeg で結合:
   ```bash
   # concat_list.txt を作成
   file '05_scene1.mp4'
   file '05_scene2.mp4'
   file '05_scene3.mp4'

   ffmpeg -y -f concat -safe 0 -i concat_list.txt -c copy 05_video_final.mp4
   ```

### デフォルト設定

| 項目 | デフォルト値 |
|---|---|
| 言語 | 日本語 (ja) |
| キャプション | `7e42d4f23fdf4a5fae97aecc5ef38436`（シンプル白文字） |
| モード | avatar4（高品質） |
| タイムアウト | 1200秒 |

### ボイスの自動選択

キャラクターの性別に応じてボイスを自動で切り替える。

| キャラクター性別 | ボイス名 | Voice ID |
|---|---|---|
| **女性** | Yuka | `Elu2nLYQkwhjIAtEOYlX8a2MSRaVOBgk` |
| **男性** | Yuma | `mrnOS0oAI096UQYY6NUGKgR8hdKyj8Ck` |

**判定ルール:**
- Step 4 で決定した CHARACTER の性別を参照する
- 性別が明示されていない場合は、ユーザーに確認する

### Topview Skill ベースパス

topview-skill スキル配下の `scripts/` ディレクトリを使用する。パスはプロジェクトルートからの相対パス `.claude/skills/topview-skill/scripts/` で解決する。

---

## Step 6: 完成動画の出力と提示

1. 結合された動画ファイルを `05_video_final.mp4` としてセッションディレクトリに保存
2. ffprobe で動画メタデータ（尺、解像度、サイズ等）を取得
3. セッションログ `session_log.md` を作成（全工程の記録）
4. ターミナルにサマリーを表示:

```
## 完成動画

ファイルパス: ./topview_output/session_XXXXXX/05_video_final.mp4

| 項目 | 内容 |
|---|---|
| 元動画 | [動画ファイル名]（モードBの場合） |
| テーマ | ... |
| 3トピック | 1. ... / 2. ... / 3. ... |
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
  ├─ 01_research_summary.md      # Step 1A: テーマ情報収集（モードA）
  ├─ 01_video_analysis.md        # Step 1B: 動画分析結果（モードB）
  ├─ 02_structure.md             # Step 2: 3トピック構成案
  ├─ 03_script_final.md          # Step 3: 確定台本
  ├─ 04_character_scene1.png     # Step 4: シーン1のキャラ画像
  ├─ 04_character_scene2.png     # Step 4: シーン2のキャラ画像
  ├─ 04_character_scene3.png     # Step 4: シーン3のキャラ画像
  ├─ 05_scene1.mp4               # Step 5: シーン1の動画
  ├─ 05_scene2.mp4               # Step 5: シーン2の動画
  ├─ 05_scene3.mp4               # Step 5: シーン3の動画
  ├─ 05_video_final.mp4          # Step 6: 結合された完成動画
  └─ session_log.md              # 全工程のログ
```

---

## 入力動画の配置

本編動画は `input_videos/` ディレクトリに配置する:

```
./input_videos/
  ├─ .gitkeep
  └─ [動画ファイル].mp4    ← ここに配置
```

---

## エラーハンドリング

| ステップ | エラー | 対応 |
|---|---|---|
| Step 1A | 情報収集失敗 | ユーザーに手動情報提供を促す |
| Step 1B | 動画分析失敗 | ファイル形式確認、リトライ |
| Step 1B | 動画が長すぎる | 最初の10分のみ分析、または要点をユーザーに確認 |
| Step 3 | セリフ文字数超過 | 自動で短縮を試み、結果を提示 |
| Step 4 | 画像生成失敗 | 3回まで自動リトライ |
| Step 5 | タイムアウト | query で再ポーリング（最大2回） |
| Step 5 | タスク失敗 | 2回まで自動リトライ |
| Step 6 | ffmpeg結合失敗 | 解像度・コーデック不一致を確認、再エンコード |
| 任意 | ユーザーが「中止」 | 現在の成果物を保存してパイプライン終了 |

---

## 制約事項

1. 既存の特定アニメ・映画キャラクターを模倣しない
2. 実在人物の顔を参照しない
3. 停止ポイント1・2で必ず停止し、承認なしに次のステップに進まない
4. 各ステップで何をしているかターミナルに逐次表示する
5. API 呼び出し回数を必要最小限にする
6. 本編動画のネタバレになりすぎないよう配慮する（モードB）
