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
- **言語設定が結果を変える**: 文字課金のモデルは1分あたりの文字数が少ない日本語で有利、
  音声トークン課金のモデルは言語によらず一定。**言語によって有利不利が逆転する**ため言語セレクタは必須
- 換算の仮定: 音声1分 = 2,400 音声トークン（OpenAI 公式の $/分 目安から逆算）、
  日本語 400字/分・英語 1,000字/分

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
