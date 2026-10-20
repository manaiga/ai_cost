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
- **Thinking 列**は「推論モードに対応しているか」の厳密な可否。既定の思考量はモデルによって異なる
- **EOL 列**は公式の deprecation / lifecycle 情報。未告知は「不明」

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
1. 各社の公式料金ページを確認
2. 現在の主要モデルと価格を特定（**期間限定価格はその終了日も**）
3. 対象ファイルのモデル構成を更新
4. EOL 到達済みのモデルを削除し、新たに判明した EOL 日を反映
5. 最終更新日（`<title>` と本文の2箇所）を更新

## 各社の公式料金ページ

### テキスト生成
- OpenAI: https://platform.openai.com/docs/pricing
- Anthropic: https://platform.claude.com/docs/en/about-claude/pricing
- Google: https://ai.google.dev/gemini-api/docs/pricing
- xAI: https://docs.x.ai/developers/pricing
- DeepSeek: https://api-docs.deepseek.com/quick_start/pricing
- AWS Bedrock: https://aws.amazon.com/bedrock/pricing/

### 画像生成
- OpenAI: https://platform.openai.com/docs/pricing
- Google: https://ai.google.dev/gemini-api/docs/pricing
- xAI: https://docs.x.ai/developers/pricing
- Black Forest Labs: https://bfl.ai/pricing
- Stability AI: https://platform.stability.ai/pricing
- Ideogram: https://about.ideogram.ai/api-pricing
- Recraft: https://www.recraft.ai/docs

### 音声
- OpenAI: https://developers.openai.com/api/docs/pricing
- Google: https://ai.google.dev/gemini-api/docs/pricing
- ElevenLabs: https://elevenlabs.io/pricing/api
- Deepgram: https://deepgram.com/pricing

## 技術仕様
- Vue.js 3 + Tailwind CSS（いずれも CDN 読み込み、ビルド不要）
- 各ページは単一の HTML ファイルで完結（共通化はしていない）
- 為替レート: [Frankfurter API](https://www.frankfurter.app/)（ECB レート）から自動取得、失敗時は ¥150 にフォールバック
