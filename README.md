# 生成AIの各社モデル別 API使用料計算機

## 概要
各社の生成AI APIの料金を比較計算するツールです。
用途ごとに料金体系が異なるため、**ページを分けて**います。

| ファイル | 対象 | 単位 |
|---|---|---|
| `index.html` | テキスト生成 | $/1M tokens（入力・出力の2軸）|
| `image.html` | 画像生成 | $/枚（解像度・品質で変動）|
| `audio.html` | 音声（読み上げ / 文字起こし / リアルタイム）| タブごとに別単位 |

各ページはヘッダーのナビゲーションで相互に行き来できます。

## モデル選定方針

### 基本方針
- **一次提供元の公式情報のみ**: 各社の公式料金ページから取得。アグリゲータ（fal / Replicate / Together 等）はホストごとに価格が異なるため掲載しない
- **コストパフォーマンス重視**: 最も安いものから主要モデル、高性能モデルまで網羅
- **一般提供のもののみ**: 限定提供（Claude Mythos、Gemini Flash Cyber 等）やサブスク専用（Midjourney）は未収録
- **EOL 到達済みは削除**: 提供終了日を過ぎたモデルは一覧から外す

### テキスト生成（index.html）
- 掲載は各社の **Standard** 価格。短コンテキスト帯（GPT-6 / GPT-5.6、Grok は 200k 未満など）に準拠
- **コンテキスト量で単価が変わるモデルは、少ない方の帯の1行だけ載せる**（行を増やさない。例: Claude Haiku 5.5 は 100k 超が5倍だが ≤100k の行のみ）
- **Thinking 列**は「推論モードに対応しているか」の厳密な可否。既定の思考量はモデルによって異なる
- **EOL 列**は公式の deprecation / lifecycle 情報。未告知は「不明」

### 絞り込み UI（3ページ共通）
- **会社チップ**: 何も選んでいない＝すべて表示。選ぶと「それだけ」、複数選ぶと「それとそれだけ」。
  「全部から一部を外す」より「これだけ見たい」が多いという前提の設計
- **最安比は表示中の中で計算**する（「Claude だけ」に絞ると Claude 内の最安が「最安」になる）
- **「各系列の最新だけ」**（index.html のみ）: 同じ系列の旧版を隠す。
  系列と版は**モデル名から自動判定**するので「最新フラグ」をデータに持たせる必要はない
  （`gpt-6.1-sol` → 系列 sol・版 6.1、`claude-opus-5-5` → opus・5.5、`gpt-4o-mini` → mini・4、`Nova 2 Lite` → lite・2）。
  括弧書きと `preview` は判定から除外。手動フラグにすると新モデル追加のたびに旧モデル側も直す必要があり事故の元になる
  - 名前から正しく判定できない例外だけ、データに `line: "..."`（系列）や `ver: 数値`（版。major×1000+minor）を書いて上書きする
  - 旧版のほうが安いことはよくある（gpt-4o-mini は gpt-5.4-mini の 1/5）ので、隠した旧版に表示中の最安より安いものがあれば警告を出す。既定は OFF（選択はブラウザに記憶）

### 画像生成（image.html）
- 掲載は **Standard** 価格、**テキストからの新規生成**。Batch は概ね半額、編集 / img2img は別料金のことがある
- **定額/枚**（xAI・Flux・Ideogram 等）と**トークン課金**（OpenAI・Gemini）が混在するため、後者は **$/枚に換算**して並べる
- **基準解像度列**が必須。同じモデルでも 1K と 4K で 2〜4倍変わるため、これを無視した比較は意味を持たない
- 品質段階で大きく動くもの（GPT Image 2.5 の medium / high / max 等）は**段階ごとに別行**にする

### 音声（audio.html）
- **課金単位が3種類混在**するため、1つの表に混ぜず**3タブに分離**している
  （同じ OpenAI 内でも `tts-1` は $/100万文字、`gpt-transcribe` は $/分、`gpt-realtime-2.1` は $/100万音声トークン）
- タブをまたいだ比較は想定しない。各タブ内で単位を統一し、比較用に **$/音声1分**へ正規化した列を持つ
- 入力は**「発話◯秒」に統一**。4つ目の「**課金方式くらべ**」タブで、同じ秒数・同じタスクを
  課金方式ごとに処理したときの差を出す（タスク内で方式別の最安モデルを1つずつ並べる）
- **言語設定が結果を変える**: 同じ原稿でも日本語は音声が長くなる（約400字/分）ため、
  音声の長さで課金される**トークン課金が日本語で割高**になる。文字課金は言語の影響を受けない
- 読み上げでは**日本語で文字課金が約1.5倍安く、英語ではトークン課金が約1.6倍安い**（向きが逆転する）。
  ただし差を過大に言わないこと。方式間の差が10%未満なら「互角（換算誤差の範囲）」と表示する実装にしている
- 課金方式の有利不利は**タスクによって逆**になる（読み上げ・文字起こしは文字/分課金が有利、
  リアルタイムはトークン課金が約4.4倍安い）
- **換算の仮定**: 音声トークン数は提供元ごとに違うので個別に持つこと。
  OpenAI 2,400/分（transcribe 系の $/分 目安から逆算、40/秒）、
  Google 1,500/分（TTS の「10秒あたり」併記から逆算、25/秒）。
  全社共通にすると Google を 1.6倍 過大計算してしまう。文字は日本語 400字/分・英語 1,000字/分
- **EOL 列が必須**: OpenAI の音声系は廃止が速い（tts-1 系は 2027-01-06、
  whisper-1 / gpt-4o-transcribe 系は 2027-02-26）。「課金方式くらべ」の代表モデルは
  **廃止予定を除外**して選ぶ
- **移行先で課金方式が変わる罠**: tts-1 の後継は `gpt-realtime-2.1-mini`（Realtime API）で、
  文字課金から音声トークン課金に変わる。日本語用途では単純置換で費用が跳ねる

### 更新手順
1. 下記「各社の公式料金ページ」と各社の deprecations ページを確認
2. 現在の主要モデルと価格を特定（**期間限定価格はその終了日も**）
3. 対象ファイルのモデル構成を更新
4. EOL 到達済みのモデルを削除し、新たに判明した EOL 日を反映
5. 最終更新日（`<title>` と本文の**2箇所とも**）を更新し、両方変わったことを確認
6. ヘッドレスブラウザで3ページを開き、計算ボタンを押して表が出ること・コンソールエラーが無いことを確認してからコミット

## 更新時のガードレール（自動更新もこれに従う）

過去に実際にやらかした失敗から作った規則。**守らないと誤った料金を公開してしまう**。

- **出典は公式ページのみ**。ブログ・アグリゲータ・比較サイトは「何か変わったかも」の手がかりに留め、数値は必ず公式で確認する。
  （FLUX の料金を二次情報から取って、存在しない `FLUX.2 [schnell]` を載せ、`FLUX.2 [pro]` を $0.05 と誤記した）
- **1回の取得で見当たらないだけでは削除しない**。公式表から**2回連続で消えていたら**削除する。
  ページの抽出が一部欠けることがあるため（`gemini-3-flash-preview` を「消えた」と削除したが実は残っていた）
- **EOL 日は「これより早くはない」の意味**。日付を過ぎても**公式表に残っていれば削除しない**。延期されることが多い。
  （gpt-4o は 2026-10-23 → 2027-01-14、Gemini 2.5 系は 10-16 から延長、Gemini 2.5 Flash Image は 10-02 → 2027-03-15。どれも日付を信じて先走った）
- **ページ読み取り結果が食い違ったら、別の取得や別ソースで確かめる**。同じページを2回読んで逆の結論が出たことがある
  （「Flash-Lite も年明けに値上げ」と一度報告したが誤りだった）
- **「自動転送」は実質の EOL として扱う**（例: 2026-12-01 以降の gemini-2.5-pro へのリクエストは 3.8 Flash に転送）
- **プロモ価格**はモデル名に終了日ラベル（例: `(〜11/30 プロモ)`）を付け、前提条件にも書く。**終了したら定価に戻してラベルを外す**
- **限定提供**（Mythos、Flash Cyber 等の審査制）・**サブスク専用**・**アグリゲータ経由**の価格は載せない
- **価格を間違って覚えていた可能性も疑う**。値下げに見えて実は元の値が誤りだったことがある（gpt-5-mini は $0.125/$1.00 と載せていたが公式は $0.25/$2.00）
- **変更が無ければコミットしない**（空コミットや日付だけの更新はしない）
- 差を過大に言わない。換算を伴う比較で 10% 未満の差は「誤差の範囲」と扱う

## 自動更新（週次）
毎週月曜 09:00（JST）に専用の Claude Code セッションが起動し、上記「更新手順」と「ガードレール」に従って
公式料金ページを確認する。変更があれば 3ページを更新して `main` にコミットし（GitHub Pages に自動反映）、
変更が無ければ何もしない。

## 各社の公式料金ページ

### テキスト生成
- OpenAI: https://developers.openai.com/api/docs/pricing
- Anthropic: https://platform.claude.com/docs/en/about-claude/pricing
- Google: https://aistudio.google.com/docs/pricing （旧 ai.google.dev/gemini-api/docs/pricing からリダイレクト）
- xAI: https://docs.x.ai/developers/pricing
- DeepSeek: https://api-docs.deepseek.com/quick_start/pricing
- AWS Bedrock: https://aws.amazon.com/bedrock/pricing/ （表が描画されないことがあるので二次情報で当たりを付けて裏取り）

### 画像生成
- OpenAI: https://developers.openai.com/api/docs/pricing
- Google: https://aistudio.google.com/docs/pricing （Imagen 4 は Vertex AI 側で、ここには無い）
- xAI: https://docs.x.ai/developers/pricing
- Black Forest Labs: https://bfl.ai/pricing
- Stability AI: https://platform.stability.ai/pricing
- Ideogram: https://about.ideogram.ai/api-pricing
- Recraft: https://www.recraft.ai/docs

### 音声
- OpenAI: https://developers.openai.com/api/docs/pricing
- Google: https://aistudio.google.com/docs/pricing
- ElevenLabs: https://elevenlabs.io/pricing/api
- Deepgram: https://deepgram.com/pricing

### 廃止（EOL）情報
- OpenAI: https://developers.openai.com/api/docs/deprecations
- Anthropic: https://platform.claude.com/docs/en/about-claude/model-deprecations
- Google: https://aistudio.google.com/docs/deprecations （料金ページ内の deprecated 注記と併せて見る。日付は「最短」で延期されることがある）
- AWS Bedrock: https://docs.aws.amazon.com/bedrock/latest/userguide/model-lifecycle.html

## 技術仕様
- Vue.js 3 + Tailwind CSS（いずれも CDN 読み込み、ビルド不要）
- 各ページは単一の HTML ファイルで完結（共通化はしていない）
- 為替レート: [Frankfurter API](https://frankfurter.dev/)（ECB レート、`api.frankfurter.dev/v1`）から自動取得、失敗時は ¥160 にフォールバックし画面に「取得失敗」と表示。旧アドレス `api.frankfurter.app` は 301 転送になりブラウザでは失敗する（これで長く ¥150 固定になっていた）。週次更新ではレートが実際に取れているかも確認する
