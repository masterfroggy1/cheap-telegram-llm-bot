# Cheap Telegram Bot with LLM Integration

A personal Telegram bot that handles everyday document, research, and productivity tasks for ~$1/month using LLM APIs — built to work entirely inside Telegram's own interface (no separate app, no separate site).

## What it does

- Works with **Word and Excel files** — reads, edits, and generates `.docx` / `.xlsx`
- Generates **PowerPoint presentations** (`.pptx`)
- **Transcribes voice messages** into clean text (filler words and noise stripped out)
- **OCR** on scanned/photographed documents (Russian + English)
- Handles **photos sent as regular images** through a vision-capable model (separate path from OCR — see architecture below)
- Small research/fact-checking tasks — e.g. verifying details for grants and competitions
- Keeps conversation context until `/new` or a restart
- Yes, it can also write you a pancake recipe

## Why Telegram

Telegram was chosen deliberately, not by default: native file handling, a built-in rich text editor for formatted replies (MarkdownV2/HTML), voice message support, and a bot API mature enough that no other platform offered a comparable combination for this use case.

## Architecture

Telegram message
      │
      ▼
 ┌─────────────────────────────┐
 │  python-telegram-bot         │  (routing, handlers)
 └─────────────┬────────────────┘
               │
      ┌────────┴─────────┐
      ▼                   ▼
 Voice message        File / Image
      │                   │
      ▼                   ▼
 Google Cloud          Scanned doc → OCR (pytesseract, rus+eng)
 Speech-to-Text         Photo/image → vision model (OpenRouter)
      │                   │
      └────────┬──────────┘
               ▼
   Documents normalized to Markdown
   (python-docx / python-pptx / openpyxl)
               │
               ▼
   Prepared context + task → LLM (DeepSeek via OpenRouter)
   + Tavily API for web search when needed
               │
               ▼
        Formatted reply in Telegram

The core design idea: do as much preprocessing as possible *before* the request hits the model — convert documents to Markdown, transcribe and clean voice input, route images to the right pipeline (OCR vs. vision) — so the LLM receives a ready, minimal context instead of raw files. This is what keeps token usage (and cost) low.

## Stack

- **Bot framework:** `python-telegram-bot`
- **LLM:** DeepSeek, via OpenRouter
- **Web search:** Tavily API (free tier)
- **Speech-to-text:** Google Cloud Speech-to-Text
- **OCR:** `pytesseract` + `Pillow` (rus+eng)
- **Documents:** `python-docx`, `python-pptx`, `openpyxl`
- **Hosting:** self-managed VPS (Hetzner)

## The Gemini dead end

The original plan was to use Gemini via Google AI Studio for the core LLM. That's not possible with a European Google account — the API isn't available for that region. Rather than spend time working around it, I switched providers entirely and moved to DeepSeek through OpenRouter, which turned out cheap enough and good enough for this use case. (The `google.generativeai` import is still sitting unused in the code — a small fossil of that first attempt.)

## Cost

No full month of steady daily use yet, so this isn't a precise measurement — but usage so far tracks at roughly **$1/month**, thanks to the preprocessing approach described above (documents converted to Markdown before hitting the model, transcription cleaned of noise, images routed to the cheapest adequate path).

## Status

Working and in daily use by me and one other person. Not polished, not "finished" — built to solve a real, specific need, and it does.
