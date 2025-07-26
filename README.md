# HscRagBot
This project is a comprehensive AI engineering system that involves building a sophisticated Retrieval-Augmented Generation (RAG) system. The system allows users to ask questions in Bengali about the story "Oporichita" by Rabindranath Tagore and receive accurate, fact-checked answers. The project culminates in a lightweight REST API for interaction.

✨ Features
Surgical Data Extraction: The system intelligently loads and processes only the relevant pages (6-17) from the source PDF, ensuring the knowledge base is clean and focused.

Robust Text Processing: Implements a word-by-word text extraction method (pymupdf) to handle potential text corruption in the source PDF file.

Advanced Hybrid Search: Utilizes a state-of-the-art EnsembleRetriever that combines the strengths of traditional keyword search (BM25) and modern semantic search (MiniLM) for high-accuracy document retrieval.

Fact-Checking LLM Chain: A precisely engineered prompt instructs the llama3-8b-8192 model via Groq to act as a fact-checker, providing direct, concise answers based only on the retrieved context.

Conversational API: Exposes the RAG system through a lightweight and easy-to-use REST API built with FastAPI.

🛠️ Technology Stack
Core Framework: LangChain

Language Model (LLM): Llama-3 8B (via Groq API)

PDF Processing: PyMuPDF

Text Embeddings: paraphrase-multilingual-MiniLM-L12-v2

Vector Database: ChromaDB

Keyword Search: rank_bm25

API Framework: FastAPI

Server: Uvicorn, Ngrok (for public tunneling)

Environment: Google Colab

🚀 Setup and Usage Guide
This project is designed to run seamlessly in a Google Colab environment.

Prerequisites
A Google Account to use Google Colab.

The source PDF file (HSC26-Bangla1st-Paper.pdf) uploaded to your Google Drive.

A Groq API Key for LLM access. You can get a free key from console.groq.com.

Instructions
Open the Notebook: Open the AI_Assessment_RAG_API.ipynb file in Google Colab.

Configure API Key:

In the Colab sidebar, click the key icon (🔑) to open the Secrets manager.

Create a new secret named GROQ_API_KEY.

Paste your Groq API key into the "Value" field.

Ensure the "Notebook access" toggle is turned on.

Update PDF Path: In Cell 3, verify that the pdf_path variable points to the correct location of your PDF file in Google Drive.

Python

# Make sure this path is correct
pdf_path = "/content/drive/MyDrive/HSC26-Bangla1st-Paper.pdf"
Run All Cells: Click Runtime -> Run all from the menu. The notebook will install all dependencies, build the RAG chain, run the tests, and finally, launch the API server.

📝 Sample Queries and Outputs
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
🔌 API Documentation
The API is built with FastAPI and provides a single endpoint for interacting with the RAG system.

Endpoint URL: /query

Method: POST

Request Body Format: application/json

Request
The request body must be a JSON object with a single key, query.

JSON

{
  "query": "আপনার প্রশ্ন এখানে লিখুন"
}
Response
The response will be a JSON object with a single key, answer.

JSON

{
  "answer": "মডেল-জেনারেটেড উত্তর এখানে আসবে"
}
Example Usage
You can use any tool like curl or Python's requests library to interact with the API. Replace YOUR_NGROK_URL with the public URL generated when you run the final cell of the notebook.

curl Example:
Bash

curl -X POST "YOUR_NGROK_URL/query" \
     -H "Content-Type: application/json" \
     -d '{"query": "বিয়ের সময় কল্যাণীর প্রকৃত বয়স কত ছিল?"}'
Expected curl Response:

JSON

{"answer":"১৫ বছর"}
Python requests Example:
Python

import requests

api_url = "YOUR_NGROK_URL/query"
payload = {"query": "কাকে অনুপমের ভাগ্য দেবতা বলে উল্লেখ করা হয়েছে?"}

response = requests.post(api_url, json=payload)

if response.status_code == 200:
    print(response.json())
else:
    print(f"Error: {response.status_code}")
Expected Python Response:

Python

{'answer': 'মামাকে'}
