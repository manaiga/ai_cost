# 生成AIの各社モデル別 API使用料計算機

## 概要
各社の生成AI APIの料金を比較計算するツールです。

## モデル選定方針

### 基本方針
- **テキスト生成に特化**: 動画生成、音声生成、画像生成は除外
- **コストパフォーマンス重視**: 最も安いものから主要モデル、高性能モデルまで網羅
- **公式情報のみ**: 各社の公式料金ページから最新情報を取得

### 各社のモデル構成
各社について以下の3つのカテゴリからモデルを選定：

1. **エントリーモデル**: 最も安価で基本的なタスク向け
2. **標準モデル**: 一般的な用途でコストと性能のバランスが良い
3. **高性能モデル**: 高品質な出力が求められるタスク向け

### 更新手順
1. 各社の公式料金ページを確認
2. 現在の主要モデルを特定
3. テキスト生成の料金（入力・出力 $/1M tokens）を取得
4. ファイルのモデル構成を更新
5. 最終更新日を更新

### 各社の公式料金ページ
- OpenAI: https://openai.com/api/pricing/
- Anthropic: https://docs.anthropic.com/en/docs/about-claude/pricing
- Google: https://ai.google.dev/gemini-api/docs/pricing
- AWS Bedrock: https://aws.amazon.com/bedrock/pricing/
- DeepSeek: https://api-docs.deepseek.com/quick_start/pricing
- Perplexity: https://docs.perplexity.ai/guides/pricing

## 技術仕様
- Vue.js 3 + Tailwind CSS
- 為替レート: 1 USD = 150 JPY（固定）
- 料金単位: $/1M tokens