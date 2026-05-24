# 📘 AWS Module 8: AI/ML and Data Analytics — Study Notes

> **Goal of this module:** Understand how AWS supports AI/ML and data analytics, from basic concepts to real-world pipelines.

---

## 🧠 Part 1: Introduction to AI and Machine Learning

### What's the difference?

```
┌─────────────────────────────────────────────────┐
│              ARTIFICIAL INTELLIGENCE             │
│   "Intelligent systems that do human-like tasks" │
│                                                  │
│   ┌──────────────────────────────────────────┐   │
│   │           MACHINE LEARNING               │   │
│   │  "Trains on data → finds patterns →      │   │
│   │   produces a MODEL → makes predictions"  │   │
│   │                                          │   │
│   │   ┌──────────────────────────────────┐   │   │
│   │   │         DEEP LEARNING            │   │   │
│   │   │  "Uses layers of neural networks │   │   │
│   │   │   that mimic the human brain"    │   │   │
│   │   │                                  │   │   │
│   │   │   ┌──────────────────────────┐   │   │   │
│   │   │   │     GENERATIVE AI        │   │   │   │
│   │   │   │  "Powered by Foundation  │   │   │   │
│   │   │   │   Models (FMs) / LLMs"   │   │   │   │
│   │   │   └──────────────────────────┘   │   │   │
│   │   └──────────────────────────────────┘   │   │
│   └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

### ML Training Flow

```
Historical Data  →  Training Process  →  ML Model  →  New Data  →  Predictions
  (patterns)          (find patterns)     (output)                  (decisions)
```

### ✅ Quick Answer
> **What does ML training produce?** An **ML Model** that can make predictions or decisions.

---

## ☁️ Part 2: AI/ML on AWS — The 3-Tier Stack

```
┌─────────────────────────────────────────────────────────────┐
│  TIER 3: ML FRAMEWORKS & INFRASTRUCTURE  (most custom)      │
│  PyTorch, TensorFlow, Apache MXNet + EC2 / EMR / ECS        │
│  → For ML engineers who need FULL control                   │
├─────────────────────────────────────────────────────────────┤
│  TIER 2: ML SERVICES                                        │
│  Amazon SageMaker AI                                        │
│  → Build, train, deploy YOUR OWN models, no infra mgmt      │
├─────────────────────────────────────────────────────────────┤
│  TIER 1: AI SERVICES  (easiest, pre-built)                  │
│  Ready-to-use managed models for specific tasks             │
│  → Language, Vision, Conversational, Personalization        │
└─────────────────────────────────────────────────────────────┘
         ↑ More skill needed          ↑ Less skill needed
```

---

## 🔧 Part 3: AWS AI Services (Tier 1)

### 🗣️ Language Services

| Service | What it does | Example Use Case |
|---|---|---|
| **Amazon Comprehend** | Extracts insights from text (sentiment, phrases, language) | Customer sentiment analysis |
| **Amazon Polly** | Text → lifelike speech | E-learning apps, accessibility |
| **Amazon Transcribe** | Speech → text | Call transcription, subtitles |
| **Amazon Translate** | Text translation across languages | Document translation |

### 👁️ Computer Vision & Search Services

| Service | What it does | Example Use Case |
|---|---|---|
| **Amazon Kendra** | NLP-powered enterprise search | Intelligent search, chatbots |
| **Amazon Rekognition** | Analyze images/videos (objects, faces, text) | Content moderation, ID verification |
| **Amazon Textract** | Extract text from documents, forms, tables | Healthcare/finance form processing |

### 💬 Conversational AI & Personalization

| Service | What it does | Example Use Case |
|---|---|---|
| **Amazon Lex** | Add voice/text chat to apps (uses NLU + ASR) | Virtual assistants, FAQ bots |
| **Amazon Personalize** | Personalized recommendations from historical data | Streaming/product recommendations |

---

## 🤖 Part 4: Generative AI on AWS

### Key Concepts

```
Foundation Model (FM)
  ├── Pre-trained on MASSIVE amounts of data
  ├── Can do MULTIPLE tasks (unlike traditional ML = 1 task)
  └── Examples: LLMs (language), image gen, music gen

Large Language Model (LLM) = type of FM trained on human language
```

### AWS Gen AI Services

```
┌─────────────────────────────────────────────────────────────┐
│  SageMaker JumpStart                                        │
│  "ML hub — deploy pre-built FMs in a few clicks"           │
│  Use when: rapid deployment, fine-tuning, prototyping       │
├─────────────────────────────────────────────────────────────┤
│  Amazon Bedrock                                             │
│  "Single API → access FMs from Amazon, Claude, Stable      │
│   Diffusion, etc. Fully managed, enterprise-grade."         │
│  Use when: building gen AI apps, multimodal content,        │
│            advanced conversational AI                       │
├─────────────────────────────────────────────────────────────┤
│  Amazon Q                                                   │
│  "AI assistant connected to YOUR company's data"           │
│                                                             │
│  Q Business → answers questions from internal docs/systems  │
│  Q Developer → code recommendations (Java, Python, JS...)   │
└─────────────────────────────────────────────────────────────┘
```

### ✅ Quick Answers
> - **Advertising agency needs text + image generation, no infra?** → **Amazon Bedrock**
> - **Dev team needs to code faster securely?** → **Amazon Q Developer**

---

## 📊 Part 5: Introduction to Data Analytics

### ETL — The Data Prep Process

```
  EXTRACT              TRANSFORM              LOAD
┌──────────┐         ┌──────────┐         ┌──────────┐
│ Pull data│ ──────► │ Clean &  │ ──────► │ Store in │
│ from     │         │ format   │         │ warehouse│
│ sources  │         │ the data │         │ or lake  │
└──────────┘         └──────────┘         └──────────┘
         automated by a DATA PIPELINE
```

> **ETL = Extract, Transform, Load**
> **Data Pipeline** = automated assembly line that makes ETL efficient and repeatable

### What is Data Analytics?
Transforming **raw historical data** to uncover **valuable insights and trends**.

Real-world use cases:
- Banks explaining loan decisions
- Medical researchers analyzing clinical trials
- Insurance companies justifying risk models

---

## 🔁 Part 6: Data Pipelines on AWS

### Full Pipeline Flow

```
[SOURCE DATA]
     │
     ▼
┌─────────────────────────────────┐
│  INGESTION                      │
│  Real-time  → Kinesis Streams   │
│  Near-real  → Data Firehose     │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  STORAGE                        │
│  Unstructured → Amazon S3       │
│  Structured   → Amazon Redshift │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  CATALOGING                     │
│  AWS Glue Data Catalog          │
│  (metadata inventory)           │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  PROCESSING (ETL)               │
│  AWS Glue   → managed ETL       │
│  Amazon EMR → big data at scale │
│              (Spark, Hadoop)    │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  ANALYSIS & VISUALIZATION       │
│  Amazon Athena    → SQL queries │
│  Amazon Redshift  → complex SQL │
│  Amazon QuickSight→ dashboards  │
│  OpenSearch       → search/logs │
└─────────────────────────────────┘
```

### Service Cheat Sheet

| Phase | Service | Key Detail |
|---|---|---|
| Ingestion (real-time) | **Kinesis Data Streams** | Serverless, auto-scales |
| Ingestion (near real-time) | **Amazon Data Firehose** | Delivers to S3/Redshift in seconds |
| Storage (data lake) | **Amazon S3** | Any data type, fully elastic |
| Storage (warehouse) | **Amazon Redshift** | Petabytes, structured/semi-structured |
| Cataloging | **AWS Glue Data Catalog** | Metadata repo, improves discovery |
| Processing | **AWS Glue** | Managed ETL jobs |
| Processing (big data) | **Amazon EMR** | Spark, Hadoop, Hive |
| Analysis | **Amazon Athena** | SQL on S3, pay per query |
| Analysis | **Amazon Redshift** | Columnar, massively parallel |
| Visualization | **Amazon QuickSight** | Dashboards for everyone |
| Search/Monitoring | **Amazon OpenSearch** | Keyword + natural language search |

---

## 🏢 Part 7: Real-World Example — E-Commerce Data Pipeline

An e-commerce company needs to serve the **same dataset** to both **data scientists** (analysis) and **ML engineers** (model training):

```
[E-Commerce App]
       │  customer data (events, transactions)
       ▼
  [Ingestion Layer]  ← Kinesis / Firehose
       │
       ▼
  [Storage Layer]  ← Amazon S3 (data lake)
       │
       ▼
  [Cataloging]  ← AWS Glue Data Catalog
       │
       ▼
  [Processing / ETL]  ← AWS Glue
       │
       ├─────────────────────────────────┐
       ▼                                 ▼
[Data Scientists]               [ML Engineers]
 Analytics & insights          Model training
 (Athena, QuickSight)          (SageMaker AI)
```

> **Key point:** One automated pipeline, multiple consumers. Everyone works from the same clean dataset.

---

## 🎯 Exam Quick-Reference

### Service → Use Case

```
Need to...                              → Use...
─────────────────────────────────────────────────────────
Analyze customer sentiment in text      → Amazon Comprehend
Convert text to speech                  → Amazon Polly
Transcribe audio calls                  → Amazon Transcribe
Translate documents                     → Amazon Translate
Search enterprise documents             → Amazon Kendra
Analyze images/videos                   → Amazon Rekognition
Extract text from forms                 → Amazon Textract
Add chatbot to app                      → Amazon Lex
Personalized recommendations            → Amazon Personalize
Build/train custom ML, no infra         → SageMaker AI
Deploy foundation models via API        → Amazon Bedrock
AI assistant from company docs          → Amazon Q Business
Faster coding with AI                   → Amazon Q Developer
Real-time data ingestion                → Kinesis Data Streams
Near-real-time ingestion to S3/Redshift → Amazon Data Firehose
Data lake storage                       → Amazon S3
Data warehouse                          → Amazon Redshift
Managed ETL                             → AWS Glue
Big data processing (Spark)             → Amazon EMR
SQL queries on S3                       → Amazon Athena
Dashboards & reports                    → Amazon QuickSight
Search + log monitoring                 → Amazon OpenSearch Service
```

---

## 🔑 Key Terms Glossary

| Term | Plain English |
|---|---|
| **AI** | Computers doing human-like tasks |
| **ML** | Subset of AI; trains on data to find patterns |
| **ML Model** | The output of ML training; used to make predictions |
| **Deep Learning** | ML with layered neural networks (mimics brain) |
| **Foundation Model (FM)** | Huge pre-trained model that does many tasks |
| **LLM** | Foundation model trained on human language |
| **Generative AI** | AI that creates new content (text, images, etc.) |
| **ETL** | Extract, Transform, Load — data prep process |
| **Data Pipeline** | Automated ETL assembly line |
| **Data Lake** | Flexible store for raw/unstructured data (S3) |
| **Data Warehouse** | Structured store optimized for analytics (Redshift) |
| **Metadata** | Data about data (used in Glue Data Catalog) |

---

