# ⚡ YOUTubeRAG: Production-Ready Conversational Engine for YouTube

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![RAG Architecture](https://img.shields.io/badge/RAG-Hybrid%20Dense%2FBM25-orange.svg)]()
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)]()

A modular, low-latency **Retrieval-Augmented Generation (RAG)** pipeline designed to ingest YouTube transcripts and audio, index timestamped semantic segments, and deliver strictly grounded answers with verifiable video citations.

Built with **hybrid search (BM25 + Dense Embeddings)**, **cross-encoder reranking**, and automated **Whisper fallback** for uncaptioned videos.

---

## 🌟 Key Features

* **Dual Ingestion Engine:** Direct API fetching via `youtube-transcript-api` with an automated fallback to **Faster-Whisper (ASR)** for videos lacking native captions.
* **Timestamp-Preserved Semantic Chunking:** Dynamically groups transcript lines using rolling token windows while maintaining granular `start` and `duration` metadata.
* **Hybrid Retrieval (Dense + Sparse):** Integrates **BM25 lexical scoring** with dense vector representations (ChromaDB / Qdrant) via Reciprocal Rank Fusion (RRF).
* **Two-Stage Reranking:** Applies a Cross-Encoder (`ms-marco-MiniLM-L-6-v2`) over top-$k$ retrieved chunks to maximize precision and suppress hallucinated context.
* **Clickable Deep-Link Citations:** Synthesizes LLM responses with inline video links formatted as `https://youtu.be/<id>?t=<seconds>`.
* **Streaming & Multi-Turn Chat:** Async streaming support with session-based memory buffering.

---

## 🏗️ System Architecture

```text
                                 ┌────────────────────────┐
                                 │   YouTube Video URL    │
                                 └───────────┬────────────┘
                                             │
                        ┌────────────────────┴────────────────────┐
                        ▼                                         ▼
            [ Captions Available? ]                    [ Captions Missing? ]
                        │                                         │
                        ▼                                         ▼
           youtube-transcript-api                    yt-dlp (Extract Audio)
                        │                                         │
                        │                                         ▼
                        │                               Faster-Whisper (ASR)
                        │                                         │
                        └────────────────────┬────────────────────┘
                                             │
                                             ▼
                               [ Semantic Rolling Chunking ]
                                (Preserves Timestamps/IDs)
                                             │
                       ┌─────────────────────┴─────────────────────┐
                       ▼                                           ▼
             [ Dense Embeddings ]                         [ Sparse Indexing ]
          (text-embedding-3-small)                             (BM25)
                       │                                           │
                       └─────────────────────┬─────────────────────┘
                                             │
                                             ▼
                                   [ Hybrid Search + RRF ]
                                             │
                                             ▼
                                  [ Cross-Encoder Reranker ]
                                  (ms-marco-MiniLM-L-6-v2)
                                             │
                                             ▼
                                     [ Prompt Engine ]
                                  + Strict Grounding Rules
                                             │
                                             ▼
                                      [ LLM Stream ]
                             (Grounded Answer + Timestamps)