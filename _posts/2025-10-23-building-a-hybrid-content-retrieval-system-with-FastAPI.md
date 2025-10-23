---
layout: post
title: "Building a Hybrid Content Retrieval System with FastAPI (MRR@3 ≥ 0.8)"
date: "2025-10-23 00:00:00 +0000"
categories: general
tags:
  [
    python,
    FastAPI,
    data science,
    embeddings,
    web scraping,
    beautifulsoup,
    requests,
    openai,
    synthetic datasets,
  ]
excerpt: "I recently applied for an ML Engineer position, and the company had the audacity to reject me (can you believe it?). Part of the application was a coding challenge process involved building a FastAPI content retrieval application that would return the top-3 blogposts from a dataset of posts that most closely matched a user's query. This post explains how I created a custom blogpost dataset, created a synthetic evaluation set, created sparse lexical features and dense semantic embeddings, and served and evalued my FastAPI locally. Everything is reproducible and there are many possibilities for extending this approach or applying it to downstream tasks."
---

## Introduction

I recently applied for an ML Engineer position, and the company had the *audacity* to reject me (can you believe it?). Part of the application was a coding challenge process involved building a FastAPI content retrieval application that would return the top-3 blogposts from a dataset of posts that most closely matched a user's query. The success criteria were:

1. Mean Reciprocal Rank at 3 (MRR@3) ≥ 0.8 when evaluated against a hidden test set of unseen `user_query` → `blogpost_title` pairs
2. deploy the app to a Google Cloud Provider (GCP) Vertex AI server to facilitate remote API calls.

The good news is that I managed to build an app that achieved the required `MRR@3` on the public test set and on a synthetic test set. The bad news was that I could not get it to run smoothly on GCP. The intricacies of cloud deployment were slightly above my pay grade ($0/hour for this exercise). Unfortunately the company didn't accept "it works on my machine" as an excuse, even when "it works on my machine" also meant it _did_ work on GCP via my machine...

The other good news is that I learned a lot about scraping HTML text data with `requests` and `beautifulsoup`, generating synthetic text datasets with `openai`, creating lexical sparse features and semantic dense embeddings (with Term Frequency-Inverse Document Frequency (TF-IDF)/BM25 and SBERT respectively), deploying a `FastAPI` app locally and on GCP with Docker, and evaluating performance.

I thought it would be worth sharing what I built and what I learned, so that hopefully someone else (maybe even me 3+ months) can learn from my mistakes. This isn't meant to be a FastAPI or embedding "primer" (there's plenty of great educational material on that already), but just to go through what I've done, and some of my design choices.

Everything here should be reproducible, and with only a few tweaks to step 1 (creating a dataset) you can easily apply this approach to any publicly available blog.

_Caveats:_

1. For the coding challenge I was provided with a dataset of ~80 blogposts, but for this demonstration I decided to do everything from scratch by first scraping blogposts from a public blog.
2. The original coding challenge dataset included blogposts that contained images, and the test sets included both image queries and text queries. Therefore, for the initial implementation I had to add image embeddings. I have decided to omit this step, and focus on working with a text-only dataset for two reasons: 1) generating a custom multimodal dataset is a significantly more complicated, 2) generating multimodal embeddings and multimodal evaluation is also significantly more complicated. By focusing on text-only datasets I was able to spend more time "productionizing" the code.

_Notes_:

- For this demonstration, the default settings use the amazing [Paul Graham blog](https://www.paulgraham.com/pypar.html). Unqualified Reservations also works as a secondary default so as to provide a very different dataset. But you can amend the code minimally to apply it to easily to any text-based blog of your choice.

## Problem & Solution Architecture

> How do you find the most relevant blogposts in a dataset of 100+ posts when given a particular text query?

A naive approach might be to choose a few keywords or key phrases of the query and `Ctrl + f` them in the entire dataset, and then return a list of the blogposts that have the highest number of occurrences of those words/phrases in their content. But how do you know what the keywords are, and how do you rank which keywords are most important, and what if the key phrases in the query aren't in the dataset at all because the query phrases the idea in one way and a most relevant blogpost phrases exactly the same idea in another way?

The explanation of the problem above already hints at the two ways to solve this problem, _lexically_ and _semantically_, which can be merged into something much more powerful.

### _Lexical_ Direct Token Matching

`Ctrl + f` is actually already an incredibly powerful technology. What would take a human several hours (even quickly scanning), can be done in microseconds even for large datasets. However, it's a very brittle approach. It only works for exact matches, has no "opinion" about which words/phrases are most relevant, and no way of ranking returns other than by count.

#### TF-IDF

Term Frequency-Inverse Document (TF-IDF) is a technique that solves these three problems and can form the bedrock of a text content retrieval solution. It really is just a fancy `Ctrl + f`. TF-IDF has been around since the [1970s](https://en.wikipedia.org/wiki/Tf–idf) (developed by [Karen Spärck](https://en.wikipedia.org/wiki/Karen_Spärck_Jones) in 1972) and basically answers the question: _"How important is a word/phrase to this document, compared to how common it is across all documents in my dataset?"_.

If you have a word _t_ in document _d_, the equation for tf-idf(_t_, _d_) is:

$$
\text{tf-idf}(t, d) = (1 + \log f_{t,d}) \times \log \frac{1 + N}{1 + \text{df}(t)} + 1
$$ where:
- $f_{t,d}$ = how many times the word in document _d_
- _N_ = the total number of documents
- df(_t_) = "document frequency", i.e. how many documents contain that word

Common words like "the" will have a high df (first term) but a low idf (second term), whereas less common words like "lisp" will have moderate/low df but high idf. The result is a large, sparse matrix  (most entries are 0) where each row represents a document and each column represents a unique token (word or phrase). When you compute the cosine similarity of this TF-IDF matrix against a vector representing the tokens of the query, you get a vector that represents how lexically similar the query is to each document. This approach can be extended to include tokens containing 2+ words, allowing you to match phrases as well as individual words.

#### BM25
BM25 (Best Match 25) is an extension of TF-IDF that accounts for the facts that repeat instances of a token in a document don't necessarily increase its relevance beyond a certain limit, and that long documents contain more unique tokens . It does this by saturating the term frequency, penalising longer documents

My implementation uses either TF-IDF or BM25, and because we chunk the documents into equally sized chunks, TF-IDF performs well. If you do not chunk and your documents are radically different lengths BM25 performs much better.

_Notes_:
- _Lexical_ just means relating to words or vocabulary, without worrying about the meaning of words. I didn't know this. Shame on me!

## _Semantic_ Meaning matching
So we have a technique that can appropriately rank documents based on their direct lexical similarity to the query, but what if we want to match the _meaning_ of the query? If we had a way to encode the _meaning_ of the query and the _meaning_ of the document we could perform cosine similarity between these encodings to evaluate how closely they matched.

Luckily for us someone has already invented such an encoding system, which encodes meaning in a high-dimensional vector space. The idea stretches back to the 1990s in distributional semantics and latent sentiment analysis, but a modern implementations is based on the Bidirectional Encoder Representation from Transformers (BERT), Sentence-BERT (SBERT).

_Notes_:
- _Semantic_ just means relating to meaning. I've never really understood the phrase _semantic meaning_ as it's a tautology ("meaning meaning").

#### SBERT
SBERT uses a siamese network architecture that encodes the query and document (or chunk) independently into dense vectors. When you encode the entire dataset, you have an SBERT matrix where each row represents a document and each column represents an axis in "idea space". For example "founding a startup" and "starting a company" only share one trivial word, but have very similar meanings, so their SBERT embeddings should have high cosine similarity (i.e. they point in the same direction in the high-dimensional "meaning" space).

If you pre-compute the SBERT embeddings for all blogposts, then at query time you would only need to compute the embedding for the query and match against the precomputed embedding matrix. This is quick and efficient.

My combining these two techniques (matching _lexical_ meaning with TF-IDF/BM25 and _semantic_ meaning with SBERT) we can solve our initial problem and quickly rank documents in a large dataset based on their similarity to the query!

## Solution
My solution (code available at [github.com/seamusbarnes/content-retriever](https://github.com/seamusbarnes/content-retriever)) is fully reproducible. The repo only contains five Python files that generate the required dataset and evaluations and then serves the FastAPI app locally. The simplified repo structure is as follows:
```bash
app/
└── main.py
scripts/
├── 01_generate_blogpost_dataset.py
├── 02_generate_evaluation_set_parallel.py
├── 03_generate_embeddings.py
└── 04_evaluate_with_stats.py
```
- `01_generate_blogpost_dataset.py`:
	- scrapes a public blog with `requests` and `beautifulsoup`, creating the dataset from scratch
	- creates and populates: `data/`, `data/html_cache/<SITE_ID>`, `data/blogposts/<SITE_ID>` and `data/analysis/<SITE_ID>`.
- `02_generate_evaluation_set_parallel.py`:
	- creates synthetic `blogpost_title` → `text_query` pairs using `openai`
	- creates and populates: `artifacts/`
- `03_generate_embeddings.py`:
	- creates sparse features (TF-IDF or BM25) and dense embeddings (SBERT) for each blogpost chunk
	- creates and populates: `artifacts/features/<HASH>`
- `main.py`:
	- FastAPI app served locally for demonstration purposes
	- loads the pre-computed artifacts
	- creates sparse features and dense embeddings from the users query and performs weighted matching against the pre-computed artifacts, returning a list of 3 blogposts most relevant to the users query
- `04_evaluate_with_stats.py`
	- iteratively prompts the FastAPI app with synthetic queries and compares the top-3 response against the "ground truth" blogpost title
	- calculates MRR@3 and logs performance metrics
### Step 1: Creating a Dataset
Raw HTML is messy as hell, so we need a mini Extract-Transform-Load (ETL) pipeline to convert this into clean, well-formatted text for downstream tasks. `01_generate_blogpost_dataset.py` achieves this using the following pipeline: discover → fetch → extract → normalise → analyse → chunk.

The core of this step is the `SiteAdapter` class (already defined for the Paul Graham blog and Unqualified Reservations HTML structure). Each `SiteAdapter` sub-class (e.g. `PaulGrahamAdapter`) has specific HTML parsing logic, and new `SiteAdapter` sub-classes can be defined for different HTML structures.

The script generates a list of URLs from the user-specified or default archive, fetches the raw HTML, and extracts the HTML into blocks using the appropriate `SiteAdapter` sub-class. The HTML and clean JSON is cached for downstream feature generation. The goal here isn't to scrape the internet, but to have a clean, reproducible local dataset that behaves the same way each run.

### Step 2: Creating a Synthetic Evaluation Set
Once we have a clean dataset of blogposts, we need a way to evaluate whether the retriever actually works. At the moment we have no way of knowing which blogpost is most relevant to a particular user query (e.g. "how should early stage founders think about scaling infrastructure" or "how do programming languages evolve over time?"). We need a "ground truth" for representative `user_query` → most relevant `blogpost_title`, against which we can benchmark our retrieval app.

One way to generate such a synthetic evaluation dataset in by using an LLM, which is surprisingly good at this kind of task. ChatGPT, with the right prompt, can, for example, generate short/medium/long length queries that relate specifically to each blogpost. This is obviously not a foolproof approach, as it might generate a query for a particular post and then later down the line come across a post for which that original query was a better match. This will add noise to our MRR@3 evaluation, but it is a very strong first pass, especially if the blogposts contain a wide variety of unique topics with minimal semantic overlap. It is also a design choice. Even if you had the author write every single evaluation set query, they would also make mistakes, maybe different mistakes, but it is worth bearing in mind when thinking about this.

The main design goals for this step were as follows:
1. scalable and safe LLM prompting that could easily generate hundreds of queries
2. deterministic file outputs
3. parallel execution for time speedup: I used Python's `futures.ThreadPoolExecutor` functionality
4. cost transparency: I didn't want to be spending more than a few 10's ¢ for the full dataset, and wanted to know how much everything costs
5. robustness: I needed things to fail smoothly, and incrementally cache outputs in case of catastrophic errors

This script creates a clean, well-structured `evaluation_data_custom_<BLOG_ID>.json` which can be used for evaluation.
### Step 3: Creating Features and Embeddings
Now that we have our blogpost dataset (from Step 1.) we need to encode that raw text into vectors that represent the "meaning" of each post. This is the "ML-core" of the project.

At a high-level, this step achieves two things:
- Represents blogpost chunks (800 token segments with 120-token overlap) as sparse lexical feature (TF-IDF or BM25) vectors and dense semantic embedding vectors (SBERT).
- Cache those pre-computed artifacts so the retrieval app API can load them quickly and compare them against test-time generated features/embeddings of user queries.

I chose to chunk the blogposts rather than creating entire-blogpost features, so that the retriever doesn't unfairly bias towards either short or long blogposts. I found that chunking improved recall, I presume because queries can match smaller, semantically coherent chunks of text rather than entire essays.

This script creates `*.npz` and `*.pkl` artifacts that need to be moved to `app/features/` for the FastAPI app to find and use them.
### Step 4: Serving a FastAPI App and Evaluating
Now that we have the two requisite components generated, the sparse and dense features over blogpost chunks, and an evaluation dataset, we can expose a tiny API that fuses the features and compares them against test-time computed query features.

#### Step 4.1: FastAPI
The FastAPI only has two endpoints:
- `/health` which returns `{"status": "OK`} and simply tells the user that the server is working
- `/predict` which accepts one or more instances of `{"text_input" : "<QUERY>"}` and returns a ranked top-K (default K=3 to match MRR@3) list of most relevant blogpost titles.

This structure compares user queries to chunks and uses the best chunk score per post (max pooling) for both lexical and semantic matching. This gives each post a single "how good was your best chunk?" result for both matching modalities. Max pooling is a design choice (and maybe a mistake in retrospect?) that means a long irrelevant blogpost but with one very relevant chunk may be ranked higher than a blogpost with higher average relevance but no standout relevant chunks.

I also used Reciprocal Rank Fusion (RRF) to fuse the lexical and semantic matching into a final score.

#### Step 4.2: Evaluation
`04_evaluate_with_stats.py` pings the API `/predict` endpoint with each short/medium/long query in our synthetic dataset, takes the returned top-3 list of blogpost titles, and compares that against the synthetic "ground-truth" blogpost title, and calculates the MRR@3. It also tracks hits and misses. For example the correct blogpost title might be returned in position 3, which is a hit but MRR@3 < 1.0, or the correct blogpost title might not be returned at all, which is a hit and MRR@3 = 0.0 for that query. At the end of the evaluation it gives the user an average MRR@3, average hit rate and some latency details. If hit rate is high but MRR@3 is mediocre, it means the app usually returns the correct blogpost, but not ranked first. Here is an example of the final result:
<div style="text-align: center;">
<img src=  "{{ site.baseurl }}/assets/posts/content-retrieval-app-evaluation.png" style="width: 100%;">
</div>

- MRR@3 = 0.9200
- Hit@3 = 95.6%

Success!

## Conclusion
Working on this project was fun. I learned a lot, and the code will live forever (not really forever) entombed in amber on [github](https://github.com/seamusbarnes/content-retriever). It works as-is, and could provide a nice jumping off point for down-stream tasks, or it could be extended significantly to incorporate blogposts containing "meaningful" images and image embeddings (how you decide to combine image and text embeddings, and what their weightings are etc. is a very interesting project), or it could be hosted in a Docker container locally or on GCP/AWS...

There's lots more here, and lots more technical detail and stretch-goal rabbit holes to go down. But for now I've had enough of this project, and am happy for the time being I can draw a line under it. If you have made it this far, thanks for reading. If you try adapting it or have any ideas for improvement, I'd love to hear about it. Any feedback, criticism, abuse, etc. is welcome.
$$
