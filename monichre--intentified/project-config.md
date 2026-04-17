---
trigger: always_on
description: analyst, exa, research
---

- **Required Rule Structure:**
  ```markdown
  ---
  description: Clear, one-line description of what the rule enforces
  globs: path/to/files/*.ext, other/path/**/*
  alwaysApply: boolean
  ---
  
[Exa home page](https://docs.exa.ai/)

Search...

**⌘K**Ask AI
- [**Exa Search**](https:/i/search)
- [**API Dashboard**](https://dashboard.exa.ai/login?redirect=/)

[**Documentation**](https://docs.exa.ai/reference/getting-started)

[**Examples**](https://docs.exa.ai/examples/exa-mcp)

[**Integrations**](https://docs.exa.ai/integrations/lang-chain-docs)

[**SDKs**](https://docs.exa.ai/sdks/python-sdk-specific*](https://docs.exa.ai/w

[**Chaocs.exa.ai/changelog/livecrawl-preferred-option)(httpscord.com/invite/HCShtBqbfV)

[**Blog**](https://exa.ai/blog)

**Live De](https://docs.exa.ai/examples/exa-mcp)
- [Hallucination Detector](https://docs.exa.ai/e- [Writing Assistant](https://docs.exa.ai/examples/demo-exa-powered-writing-assistant)
- [Chat app](https://chat.exa.ai/)
- [Company researcher](https://companyresearcher.exa.ai/)

**Tutorials**
- [Hacker News Clone](https://docs.exa.ai/examples/live-demo-hacker-news-clone)
- [B://docs.exa.ai/examples/recent-news-summarizer)
- [Building a Hallucination Checker](https://docs.exa.ai/examples/identifying-hallucns-with-exa)
- [RAG Q&A](https://docs.exa.ai/examples/exa-rag)
- [**Company Analyst**](https://docs.exa.ai/examples/company-analyst)
- [Exa Researcher - JavaScript](https://docs.exa.ai/examples/exa-researcher)
- [Exa Researcher - Python](https://docs.exa.ai/examples/exa-researcher-python)
- [Recruiting Agent](https://docs.exa.ai/examples/exa-recruiting-agent)
- [Phrase Filters: Niche Company Finder](https://docs.exa.ai/examiche-company-finder-with-phrase-filters)
- [Job Search with Exa](https://docs.exa.ai/examples/job-search-with-exa)
- [Build a Retrieval Agent with LangGraph](https://docs.exa.ai/examples/getting-started-with-rag-in-langgraph)
- Instructor](https://docs.exa.ai/examples/getting-started-with-exa-in-instructor)

**Tutorials**

# **Company Analyst**

Example project using the Exa Python SDK.

---

## [**​**](https://docs.exa.ai/examples/company-analyst#what-this-doc-covers)

**What this doc covers**
1. Using Exa’s link similarity search to find related links
2. Using the keyword search setting with Exa search_and_contents

---

In this example, we’ll build a company analyst tool that researches companies relevant to what you’re interested in. If you just want to see the code, check out the [**Colab notebook**](https://colab.research.google.com/drive/1VROD6zsaDh%5FrSmogSpSn9FJCwmJO8TSi?here).

The code requires an [**Exa API key**](https://dashboard.exa.ai/api-keys) and an [**OpenAI API key**](https://platform.openai.com/api-keys). Get 1000 free Exa searches per month just for [**signing up**](https://dashboard.exa.ai/overview)!

## [**​**](https://docs.exa.ai/examples/company-analyst#shortcomings-of-google)

**Shortcomings of Google**

Say we want to find companies similar to [**Thrifthouse**](https://thrift.house/), a platform for selling secondhand goods on college campuses. Unfortunately, googling “[**companies similar to Thrifthouse**](https://www.google.com/search?q=companies+similar+to+Thrifthouse)” doesn’t do a very good job. Traditional search engines rely heavily on keyword matching. In this case we get results about physical thrift stores. Hm, that’s not really what I want.

Let’s try again, this time searching based on a description of the company, like by googling “[**community based resale apps**](https://www.google.com/search?q=community+based+resale+apps).” But, this isn’t very helpful either and just returns premade SEO-optimized listicles…

What we really need is neural search.

## [**​**](https://docs.exa.ai/examples/company-analyst#what-is-neural-search%3F)

**What is neural search?**

Exa is a fully neural search engine built using a foundational embeddings model trained for webpage retrieval. It’s capable of understanding entity types (company, blog post, Github repo), descriptors (funny, scholastic, authoritative), and any other semantic qualities inside of a query. Neural search can be fan traditional keyword-based searches for these complex queries.

## [**​**](https://docs.exa.ai/examples/company-analnding-companies-with-exa-link-similarity-search)

**Finding companies with Exa link similarity search**

Let’s try Exa, using the Python SDK! We can use the`find_similar_and_contents` function to find similar links and get contents from each link. The input is simply a URL, [**https://thrift.house**](https://thrift.house/) and we set `num_results=10`(this is customizable up to thousands of results in Exa).

By specifying `highlights={"num_sentences":2}` for each search result, Exa will also identify and return a two sentence excerpt from the content that’s relevant to our query. This will allow us to quickly understand each website that we find.

**Python**

**Copy**

**Ask AI**

```
!pip install exa_py

from exa_py import Exa
import os

EXA_API_KEY= os.environ.get("EXA_API_KEY")

exa = Exa(api_key=EXA_API_KEY)
input_url = "https://thrift.house"

search_response = exa.find_similar_and_contents(
        input_url,
        highlights={"num_sentences":2},
        num_results=10)

companies = search_response.results

print(companies[0])
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Monichre) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
