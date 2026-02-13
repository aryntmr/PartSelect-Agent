# PartSelect AI Agent

An intelligent AI agent that helps users find the right appliance parts using hybrid SQL and vector search capabilities.

## Overview

This project implements a conversational AI agent powered by Claude Sonnet 4.5 that can answer questions about appliance parts for refrigerators and dishwashers. The agent intelligently routes queries between structured SQL database searches and semantic vector searches to provide accurate, relevant results.

## Architecture

**Hybrid Search System:**
- **SQL Tool**: Handles structured queries (price comparisons, filtering by brand/rating, compatibility checks)
- **Vector Search Tool**: Handles semantic queries (symptom-based search, part recommendations, natural language descriptions)

**Agent Framework:**
- LangGraph ReAct agent with tool-calling capabilities
- Automatic query routing based on question type
- Streaming responses with real-time updates

## Tech Stack

**Backend:**
- FastAPI
- LangGraph
- Claude Sonnet 4.5 (claude-sonnet-4-5-20250929)
- PostgreSQL (normalized 3NF schema)
- FAISS vector database
- sentence-transformers (all-MiniLM-L6-v2)

**Frontend:**
- React
- rsuite UI components
- framer-motion for animations
- marked for markdown rendering

**Data Pipeline:**
- Selenium + BeautifulSoup for web scraping
- pandas for data processing
- ThreadPoolExecutor for parallel enrichment

## Database Schema

**parts** table (92 parts):
- part_id, part_number (PS number), manufacturer_part_number
- part_name, brand, appliance_type
- current_price, original_price, has_discount (generated), discount_percentage (generated)
- rating, review_count
- description, symptoms, replacement_parts
- installation_difficulty, installation_time
- image_url, product_url, video_url

**models** table (1,845 models):
- model_id, model_number, brand, model_url

**part_model_mapping** table (2,735 mappings):
- Links parts to compatible appliance models
- Foreign key constraints ensure referential integrity

## Setup Instructions

### 1. Backend Setup

```bash
cd backend
pip install -r requirements.txt

# Set environment variables
export ANTHROPIC_API_KEY="your_api_key_here"
export DATABASE_URL="postgresql://user:password@localhost/partselect"

# Initialize database
psql -U postgres -f database/schema.sql
python database/load_data.py

# Start backend server
uvicorn main:app --reload --port 8000
```

### 2. Frontend Setup

```bash
cd frontend
npm install
npm start
```

Frontend runs on http://localhost:3000

### 3. Data Scraping (Optional)

To scrape fresh data from PartSelect:

```bash
cd scraping

# Step 1: Scrape initial parts list (fast)
python scrape_parts.py

# Step 2: Enrich with detailed data (slow - uses Selenium)
python enrich_parts.py

# Step 3: Load into database
cd ../database
python load_data.py
```

**Note:** Scraping is configured for 60 refrigerator parts + 60 dishwasher parts. Adjust `PARTS_PER_CATEGORY` in `scrape_parts.py` if needed.

## Data Quality

Current dataset includes:
- **92 appliance parts** (refrigerators and dishwashers)
- **41 unique rating values** (not all 5.0 - real ratings from PartSelect)
- **18 General Electric parts** (full brand names, not abbreviations)
- **1,845 compatible models**
- **2,735 part-model compatibility mappings**

All manufacturer part numbers are real (e.g., WD21X31902C, DA97-15217D) - not PartSelect PS numbers.

## Key Features

### SQL Search Tool
- Filter by price range, brand, appliance type, rating
- Find parts for specific model numbers
- Compare prices and ratings
- Get parts with video installation guides

### Vector Search Tool
- Semantic search based on problem descriptions ("my ice maker is leaking")
- Find parts by symptoms
- Similarity-based recommendations
- Handles natural language queries

### Agent Capabilities
- Automatic tool selection based on query type
- Multi-step reasoning for complex questions
- Streaming responses with thinking process visible
- Error handling and retry logic

## Project Structure

```
.
├── backend/
│   ├── agent/              # LangGraph agent implementation
│   ├── database/           # Schema and data loading scripts
│   ├── models/             # Pydantic models
│   ├── routes/             # FastAPI routes
│   ├── services/           # Database and vector services
│   ├── tools/              # SQL and vector search tools
│   └── main.py
├── frontend/
│   └── src/
│       ├── components/     # React components
│       └── api/            # API client
├── scraping/
│   ├── scrape_parts.py     # Initial scraper (fast)
│   ├── enrich_parts.py     # Detail scraper (slow)
│   └── utils/              # Data cleaning utilities
└── data/
    └── processed/          # CSV files with scraped data
```

## Example Queries

**SQL Tool queries:**
- "Show me all parts under $50"
- "Find parts for model number MFI2570FEZ"
- "What are the highest rated dishwasher parts?"

**Vector Search queries:**
- "My ice maker is not making ice"
- "Water is leaking from the bottom of my refrigerator"
- "Find parts similar to ice maker assemblies"

## Strengths

1. **Intelligent Query Routing**: Hybrid architecture automatically chooses the right search method (SQL vs Vector) based on query type
2. **Rich Data Quality**: Detailed part information including symptoms, installation difficulty, compatible models, and video guides

## Weaknesses

1. **Scraping Fragility**: Web scraper depends on PartSelect's HTML structure - site changes will break extraction logic
2. **Limited Dataset Size**: Only 92 parts across 2 appliance types - not production-scale coverage

## License

Educational assessment project.
