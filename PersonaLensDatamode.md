# 📊 InsightLens Data Model

This document defines the data model for InsightLens, a natural-language-powered insight engine that retrieves structured insights from open web sources like news, social media, and forums.

---

## 🧠 1. UserQuery

Stores the original natural language query input by the user.

| Field         | Type      | Description                              |
|--------------|-----------|------------------------------------------|
| id           | UUID      | Primary key                              |
| user_id      | UUID      | Foreign key to `UserProfile`             |
| query_text   | TEXT      | Raw natural language input               |
| timestamp    | DATETIME  | Query submission time                    |

---

## 💡 2. ParsedIntent

Stores parsed components from the query used to drive downstream search and analysis.

| Field         | Type      | Description                                     |
|---------------|-----------|-------------------------------------------------|
| id            | UUID      | Primary key                                     |
| query_id      | UUID      | Foreign key to `UserQuery`                      |
| topic         | TEXT      | Extracted entity or topic of interest           |
| intent_type   | TEXT      | Intent classification (e.g. sentiment, persona) |
| time_range    | TEXT      | Parsed timeframe (e.g., "last 3 months")        |
| region        | TEXT      | Parsed geography (if applicable)                |

---

## 🌐 3. DataSource

Defines a third-party or open web source used for data ingestion.

| Field       | Type     | Description                        |
|------------|----------|------------------------------------|
| id         | UUID     | Primary key                        |
| name       | TEXT     | Source name (e.g., Twitter, Reddit)|
| type       | TEXT     | API, scraper, webhook, etc.        |
| url        | TEXT     | Source base URL                    |

---

## 📥 4. RawMention

Raw text and metadata captured directly from the data source.

| Field         | Type      | Description                           |
|--------------|-----------|---------------------------------------|
| id           | UUID      | Primary key                           |
| source_id    | UUID      | Foreign key to `DataSource`           |
| timestamp    | DATETIME  | When the mention was posted           |
| author       | TEXT      | Username or anonymized author ID      |
| text         | TEXT      | Raw post or comment text              |
| url          | TEXT      | Link to the original post             |

---

## 🔍 5. EnrichedMention

Processed and structured version of `RawMention`, used for analysis.

| Field             | Type      | Description                             |
|------------------|-----------|-----------------------------------------|
| id               | UUID      | Primary key                             |
| raw_mention_id   | UUID      | Foreign key to `RawMention`             |
| cleaned_text     | TEXT      | Normalized/processed text               |
| language         | TEXT      | Detected language                       |
| sentiment_id     | UUID      | Foreign key to `SentimentScore`         |

---

## 😃 6. SentimentScore

Stores model-generated sentiment for a mention.

| Field         | Type     | Description                            |
|--------------|----------|----------------------------------------|
| id           | UUID     | Primary key                            |
| score        | FLOAT    | Sentiment polarity (-1.0 to 1.0)       |
| label        | TEXT     | "positive", "neutral", or "negative"   |
| model_used   | TEXT     | Model version used                     |

---

## 🏷️ 7. EntityTag

Stores extracted named entities (e.g., brands, people, topics).

| Field          | Type     | Description                          |
|----------------|----------|--------------------------------------|
| id             | UUID     | Primary key                          |
| mention_id     | UUID     | Foreign key to `EnrichedMention`     |
| entity_type    | TEXT     | Type of entity (brand, person, etc.) |
| value          | TEXT     | The recognized entity text           |
| confidence     | FLOAT    | Confidence score (0 to 1)            |

---

## 🧵 8. NarrativeCluster

Groups mentions into thematic clusters (e.g., affordability, innovation).

| Field         | Type     | Description                               |
|--------------|----------|-------------------------------------------|
| id           | UUID     | Primary key                               |
| query_id     | UUID     | Foreign key to `UserQuery`                |
| label        | TEXT     | Descriptive label for the cluster         |
| keywords     | TEXT[]   | Top keywords defining the cluster         |
| mentions     | UUID[]   | Array of `EnrichedMention` IDs            |

---

## 🧬 9. PersonaCluster

Segments of users/personas derived from behavior and language patterns.

| Field         | Type     | Description                                   |
|--------------|----------|-----------------------------------------------|
| id           | UUID     | Primary key                                   |
| query_id     | UUID     | Foreign key to `UserQuery`                    |
| label        | TEXT     | Descriptive name (e.g., “Fitness Moms”)       |
| demographics | JSON     | Inferred attributes (age, region, lifestyle)  |
| keywords     | TEXT[]   | Representative keywords used by this persona |

---

## 👤 10. UserProfile (Optional)

User metadata for personalization or saved preferences.

| Field         | Type      | Description                       |
|--------------|-----------|-----------------------------------|
| id           | UUID      | Primary key                       |
| email        | TEXT      | Email address                     |
| name         | TEXT      | User's full name                  |
| created_at   | DATETIME  | Account creation time             |
| preferences  | JSON      | Saved filters, formats, settings  |

---

## 🔁 Example Data Flow

```text
UserQuery ──▶ ParsedIntent ──▶ DataSource
                                  │
                          (feeds into)
                                  ▼
                            RawMention ──▶ EnrichedMention
                                  │                │
                      ┌───────────┴──────┐         ├────▶ SentimentScore
                      ▼                  ▼         └────▶ EntityTag
              NarrativeCluster      PersonaCluster
