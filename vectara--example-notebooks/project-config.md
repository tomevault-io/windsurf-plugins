---
trigger: always_on
description: Vectara is an end-to-end platform designed to empower product builders by embedding powerful Generative AI features into their applications.
---

# Vectara API
Vectara is an end-to-end platform designed to empower product builders by embedding powerful Generative AI features into their applications.
Vectara supports an API enabling full access to its features primarily focused around Retrieval Augmented Generation (RAG) capabilities, allowing businesses to integrate conversational AI and question-answering functionalities safely and affordably.

Vectara supports REST API endpoints, using GET and POST requests for the most common operations.
Use APIv2 only (do not use APIv1)

It follows HTTP status code conventions (2xx indicating a successful response, 4xx indicating an error in the request, and 5xx indicating a server error).
In the case of a 4xx error, the response object will likely include a "messages" key, which is a list of strings indicating the errors of the request.

You can create a corpus in Vectara that would hold your data, or use an existing corpus if you know the corpus key.

The most common API endpoints used are the query APIs (for single or multiple corpora), the index and file upload APIs (for adding new documents to corpora), the document APIs (for retrieving, listing, and summarizing documents in a corpus), the corpus APIs (for retrieving metadata about a corpus), and the factual consistency APIs.

Every request will require an API key as a credential in the header of the request.
You will also need to specify the corpus key(s) for the corpora that you want to perform the operation on.

## Metadata

Many API endpoints have a metadata_filter argument which allows you to filter the documents by their associated metadata.
Vectara has a specific structure for how these filters should be expressed.

This page contains an overview about the different types of metadata filters: https://docs.vectara.com/docs/learn/metadata-search-filtering/filter-overview

Each document has some default metadata filters that are automatically produced. You can find them here: https://docs.vectara.com/docs/learn/metadata-search-filtering/ootb-metadata-filters

There is a fixed set of functions and operators that can be used with metadata filter expressions, which you can learn about on this page: https://docs.vectara.com/docs/api-reference/search-apis/sql/func-opr

## Query APIs

The query endpoints allow you to perform RAG across one or more corpora in a user account, retrieving the most relevant documents and generating a response to the query using an LLM.
The API request includes 2 main groups of parameters:

1.  Search parameters that allow you to filter data by associated document or part metadata, a hybrid search parameter (how much to use keyword vs. neural search), how to rerank results after the initial retrieval, and how much additional context to provide with the matching texts.
2.  Summarization parameters that control the generated response by the LLM, including the LLM model, a custom prompt, the maximum number of search results that should be used to generate the response, the response language, how to present citations, and whether to include the factual consistency score with the query results (Factual Consistency Score is a metric that evaluates the likelihood of AI-generated summaries being factually accurate based on the retrieved data; it is available for responses generated in English, German, French, Portuguese, Spanish, Arabic, Chinese-Simplified, Korean, Russian, Japanese, and Hindi).

Custom prompts must follow a specific template which is specified on this page: https://docs.vectara.com/docs/prompts/vectara-prompt-engine.

When creating these prompts, you can access certain variables and functions which can be found on this page: https://docs.vectara.com/docs/prompts/custom-prompts-with-metadata

If you want to query a single Vectara corpus, you should use the Advanced Single Corpus Query endpoint.
The documentation for this endpoint can be found on this page: https://docs.vectara.com/docs/rest-api/query-corpus

If you want to query multiple corpora at once, use the Multiple Corpora Query endpoint. Documentation is available here: https://docs.vectara.com/docs/rest-api/query

## Index and File Upload APIs

If you need to add a new document to a Vectara corpus, you can use the Index and File Upload API endpoints.

The File Upload API allows you to upload PDF and Microsoft Word files as well as other text documents.
The supported file types can be found on this page: https://docs.vectara.com/docs/api-reference/indexing-apis/file-upload/file-upload-filetypes

Vectara will handle the chunking and indexing of these documents for you so that it is optimized for high-quality retrieval.
The documentation for this endpoint can be found on this page: https://docs.vectara.com/docs/rest-api/upload-file

If you have all of the text for a document and want more control over how the document is represented in the Vectara corpus,
you can index the document using the add document API. This supports two kinds of documents:
1.  A structured document where you provide the structure of the document (such as the different sections and subsections of the document).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vectara/example-notebooks](https://github.com/vectara/example-notebooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
