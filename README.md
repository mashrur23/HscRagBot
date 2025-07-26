# HscRagBot
This project is a comprehensive AI engineering system that involves building a sophisticated Retrieval-Augmented Generation (RAG) system. The system allows users to ask questions in Bengali about the story "Oporichita" by Rabindranath Tagore and receive accurate, fact-checked answers. The project culminates in a lightweight REST API for interaction.

## ✨ Features

Surgical Data Extraction: The system intelligently loads and processes only the relevant pages (6-17) from the source PDF, ensuring the knowledge base is clean and focused.

Robust Text Processing: Implements a word-by-word text extraction method (pymupdf) to handle potential text corruption in the source PDF file.

Advanced Hybrid Search: Utilizes a state-of-the-art EnsembleRetriever that combines the strengths of traditional keyword search (BM25) and modern semantic search (MiniLM) for high-accuracy document retrieval.

Fact-Checking LLM Chain: A precisely engineered prompt instructs the llama3-8b-8192 model via Groq to act as a fact-checker, providing direct, concise answers based only on the retrieved context.

Conversational API: Exposes the RAG system through a lightweight and easy-to-use REST API built with FastAPI.

## 🛠️ Technology Stack

Core Framework: LangChain

Language Model (LLM): Llama-3 8B (via Groq API)

PDF Processing: PyMuPDF

Text Embeddings: paraphrase-multilingual-MiniLM-L12-v2

Vector Database: ChromaDB

Keyword Search: rank_bm25

API Framework: FastAPI

Server: Uvicorn, Ngrok (for public tunneling)

Environment: Google Colab

## 🚀 Setup and Usage Guide

This project is designed to run seamlessly in a Google Colab environment.

Prerequisites

1. A Google Account to use Google Colab.

2. The source PDF file (HSC26-Bangla1st-Paper.pdf) uploaded to your Google Drive.

3. A Groq API Key for LLM access. You can get a free key from console.groq.com.

Instructions

1. Open the Notebook: Open the AI_Assessment_RAG_API.ipynb file in Google Colab.

2.  Configure API Key:

* In the Colab sidebar, click the key icon (🔑) to open the Secrets manager.

* Create a new secret named GROQ_API_KEY.

* Paste your Groq API key into the "Value" field.

* Ensure the "Notebook access" toggle is turned on.

3. Update PDF Path: In Cell 3, verify that the pdf_path variable points to the correct location of your PDF file in Google Drive.
   
```python

# Make sure this path is correct
pdf_path = "/content/drive/MyDrive/HSC26-Bangla1st-Paper.pdf"
```


4. Run All Cells: Click Runtime -> Run all from the menu. The notebook will install all dependencies, build the RAG chain, run the tests, and finally, launch the API server.

## 📝 Sample Queries and Outputs
Here are some sample interactions with the RAG system.

Bengali Queries

Query 1:

অনুপমের ভাষায় সুপুরুষ কাকে বলা হয়েছে?

Expected Response:

শম্ভুনাথ

Query 2:

কাকে অনুপমের ভাগ্য দেবতা বলে উল্লেখ করা হয়েছে?

Expected Response:

মামাকে

Query 3:

বিয়ের সময় কল্যাণীর প্রকৃত বয়স কত ছিল?

Expected Response:

১৫ বছর

English Query Example

While the system is optimized for the Bengali source text, it can understand and respond to English questions about the content.

Query:

Who is the main guardian of Anupam?


Expected Response:

মামা


## 🔌 API Documentation

The API is built with FastAPI and provides a single endpoint for interacting with the RAG system.

* Endpoint URL: /query

* Method: POST

* Request Body Format: application/json

Request

The request body must be a JSON object with a single key, query.

```JSON

{
  "query": "আপনার প্রশ্ন এখানে লিখুন"
}
```

Response

The response will be a JSON object with a single key, answer.

```JSON

{
  "answer": "মডেল-জেনারেটেড উত্তর এখানে আসবে"
}
```

Example Usage

You can use any tool like curl or Python's requests library to interact with the API. Replace YOUR_NGROK_URL with the public URL generated when you run the final cell of the notebook.

curl Example:
```Bash

curl -X POST "YOUR_NGROK_URL/query" \
     -H "Content-Type: application/json" \
     -d '{"query": "বিয়ের সময় কল্যাণীর প্রকৃত বয়স কত ছিল?"}'
```

Expected curl Response:

```JSON

{"answer":"১৫ বছর"}
```

Python requests Example:

```Python

import requests

api_url = "YOUR_NGROK_URL/query"
payload = {"query": "কাকে অনুপমের ভাগ্য দেবতা বলে উল্লেখ করা হয়েছে?"}

response = requests.post(api_url, json=payload)

if response.status_code == 200:
    print(response.json())
else:
    print(f"Error: {response.status_code}")
```

Expected Python Response:

```Python

{'answer': 'মামাকে'}
```

## 🧠 Project Questions & Engineering Decisions

This section details the thought process and technical decisions made during the development of this RAG system.

**1. What method or library did you use to extract the text, and why? Did you face any formatting challenges?**

I used the PyMuPDF library for text extraction. Initially, I faced significant formatting challenges; the standard get_text() method resulted in a heavily corrupted and jumbled text layer, making it unusable for any reliable retrieval.

To overcome this, I switched to a more robust, low-level extraction method: page.get_text("words"). This approach extracts the text word by word and then reconstructs the sentences. This was a crucial decision because it proved to be far more resilient to the PDF's flawed font encoding and structure, providing a much cleaner (though still not perfect) text base to build upon.

---

**2. What chunking strategy did you choose and why do you think it works well?**

I chose a RecursiveCharacterTextSplitter with a chunk_size of 512 characters and an overlap of 128 characters.

This strategy was a deliberate choice for several reasons:

* Context-Awareness: A chunk size of 512 is large enough to contain one or two complete sentences, which is vital for the embedding model to understand the semantic context of the text.

* Precision: It's small enough to ensure that the retrieved chunks are highly focused and don't contain too much irrelevant noise from surrounding paragraphs, which was a major issue with this particular document.

* Balance: This provides the best balance between providing sufficient context for semantic meaning and maintaining precision, which is key for a successful RAG pipeline.

---

**3. What embedding model did you use and why did you choose it?**

I used the paraphrase-multilingual-MiniLM-L12-v2 model. This was a strategic decision made after experimentation. While larger models exist, this one was chosen for its optimal balance of performance, speed, and robustness.

Specifically, its multilingual capabilities and smaller, more general architecture made it surprisingly effective at handling the noisy, partially corrupted Bengali text from the source PDF. It was less sensitive to the minor errors in the text compared to larger, higher-precision models, which tended to "over-analyze" the noise.

---

**4. How are you comparing the query with your stored chunks? Why this choice?**

I am using a Hybrid Search strategy, implemented with LangChain's EnsembleRetriever. This method combines the results from two different search techniques:

* Semantic Search (ChromaDB): This uses the embedding model to find chunks that are conceptually similar to the user's query. It's great for understanding the user's intent.

* Keyword Search (BM25): This is a classic algorithm that finds chunks containing the exact words from the query. It's powerful for matching specific names, places, or terms.

I chose this hybrid approach because it's the most robust solution for this project. The semantic search helps find relevant context even if the wording is different, while the keyword search ensures that we don't miss chunks that contain important proper nouns. This combination makes the retrieval process significantly more reliable than relying on a single method alone.

---

**5. How do you ensure meaningful comparison? What happens if the query is vague?**

Meaningful comparison is ensured directly by the Hybrid Search mechanism. By looking at both the semantic meaning (what the user is asking about) and the lexical keywords (the specific words they use), the system gets a much more holistic understanding of the query.

If a query is vague (e.g., "tell me about the story"), the system's performance would gracefully degrade but not completely fail. The semantic search component would likely retrieve chunks that are central to the story's main themes. However, the keyword search would be less effective. The LLM would then receive a very broad context, and its answer would likely be a general summary rather than a specific fact, which is the expected and appropriate behavior for a vague query.

---

**6. Do the results seem relevant? If not, what might improve them?**

After implementing the Hybrid Search on the word-by-word extracted text, the results became significantly more relevant and accurate. However, the system is still not perfect due to the fundamental limitations of the source material.

The single greatest improvement would not come from a better model or a different retrieval strategy, but from acquiring a clean, non-corrupted source document. The core bottleneck of this entire project has been the poor quality of the text extracted from the provided PDF. With a perfect text source, the current architecture would perform with extremely high accuracy.
