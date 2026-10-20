# NewsFindr: Personalized News Retrieval Agent

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://github.com/Sachin-Abishek/news_findr/blob/main/NewsFindr.ipynb)

An agentic AI system that retrieves news matched to a user's saved interests. It looks up the user's interests in a database, builds search queries, searches the web, filters results by source credibility, and returns short summaries with links to the top three sources.

## Problem
Generic news feeds and plain keyword search cause information overload and make it hard to find trustworthy, relevant content. NewsFindr aims to deliver current, personalized news from credible outlets, without outdated or irrelevant content.

## Objectives
- Provide real-time, personalized news retrieval based on user interests
- Prefer trusted sources and reduce misinformation
- Give concise summaries with links instead of an information dump
- Remove outdated and irrelevant results

## How it works

```
User email ─► SQL Agent (fetch interests) ─► ReAct Agent
                                                 │
              ┌──────────────────────────────────┤
              ▼                ▼                 ▼                ▼
      ExpandSearchQueries  DuckDuckGoSearch  CredibilityFilter  SummarizeNews
      (LLM builds queries) (top 10 results)  (LLM rates sources) (summary + URLs)
```

1. **SQL agent**: converts a natural-language request into SQL and fetches the user's interests from `customer.db` by email (e.g. `["India", "Politics", "Science"]`). It also handles unknown emails by returning "no records found".
2. **ExpandSearchQueries tool**: uses the LLM to turn each interest and the user's question into a precise news search query.
3. **DuckDuckGoSearch tool**: runs the query using the `ddgs` library and returns title, URL and snippet for up to 10 results.
4. **CredibilityFilter tool**: asks the LLM to rate each result for credibility, reliability and relevance.
5. **SummarizeNews tool**: summarizes the top 3 selected sources and returns the summary together with their URLs.
6. **Agent**: a LangChain zero-shot ReAct agent decides which tool to call at each step.

Two LLM instances are set up (`temperature=0.7` and `temperature=0`). The deterministic one (`temperature=0`) is used for the agent because factual, consistent output matters for news.

## Tech stack
Python, LangChain, Groq API (`meta-llama/llama-4-scout-17b-16e-instruct`), SQLite, `ddgs` (DuckDuckGo search), pandas, pydantic

## Data
`customer.db` is a SQLite database with user records: `id`, `customer_id`, `name`, `email`, `interests` (a JSON list) and `last_updated`. It holds 15 unique user emails in this project.

## Example queries
Run for a user whose interests are India, Politics and Science:

| Query | What the agent returned |
|---|---|
| "What are the latest updates and breaking stories in India?" | Key themes (politics, business, technology, sports, health, environment) with links to BBC News, The Hindu and Google News |
| "What are the recent news about Science in India?" | Themes such as space exploration, research, emerging technology and government initiatives, with links to The Hindu, Hindustan Times and NDTV |
| "How are politics and science shaping India lately?" | Themes on the state of India's science ecosystem and politicization of science, with links to The Indian Express, NYT and Scientific American |

## How to run
1. Click the **Open in Colab** badge above.
2. Add your Groq API key in Colab **Secrets** as `GROQ_API_KEY` and enable notebook access.
3. Upload `customer.db` to `/content/`.
4. Run the install cells (restart the runtime if Colab asks you to), then run the remaining cells.
5. Call `query_response(email, user_query)` with a user's email and a question:
   ```python
   query_response("george.5483cb53-8@gmail.com", "What are the recent news about Science in India?")
   ```

## Limitations
- **Summaries are not read from the articles.** The summarizer receives only URLs, so its output reflects general knowledge of each site rather than the actual stories. Fetching article text before summarizing is the main fix needed.
- **Mostly section home pages.** Search returns site and section pages more often than individual articles, which limits how "breaking" the results are.
- **Result quality varies by query.** The third query returned some older or off-topic results (e.g. a 2021 article, Quora, ILO) and the credibility filter did not clearly remove them.
- **LLM-based credibility scoring is inconsistent.** It can rate the same outlet differently in different places and is not a substitute for a maintained source-reliability list.
- **Tool input handling is fragile.** The query-expansion tool received malformed input in several runs and the agent worked around it by writing its own queries.
- **No user verification.** Interests are fetched with just an email address.
- **Dependency conflicts.** Pinned older LangChain versions show pip conflict warnings in the install output.

## Future improvements
- Fetch and parse full article text, then summarize from that
- Use a news search API (or DuckDuckGo news search) with date filters
- Replace LLM credibility scoring with an allow-list or scored source database
- Validate tool inputs with Pydantic schemas
- Add user authentication and a simple UI

## Author
**Sachin**: [LinkedIn](<your-linkedin-link>) | [GitHub](https://github.com/<your-username>)
