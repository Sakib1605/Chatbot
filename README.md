## 🌍 Travel Assistant Chatbot
### A RAG-based AI Assistant built with LangChain, Pinecone, HuggingFace Embeddings & Flask

This project is an end-to-end Travel Assistant Chatbot that uses Retrieval-Augmented Generation (RAG) to answer user questions based on a curated travel knowledge base.
It combines semantic search, LLM reasoning, and a simple Flask/Gradio frontend to deliver accurate, grounded responses without hallucinations.

## 🚀 Features

- 🔍 RAG Pipeline using LangChain

- 📄 PDF ingestion → clean text extraction

- ✂️ Recursive text splitting for high-quality embeddings

- 🧠 HuggingFace Sentence Transformer embeddings

- 📚 Pinecone vector database for semantic search

- 🤖 GPT-4o-mini (or any OpenAI model) for final generation

- 🧭 Travel-focused knowledge base (tourism, cities, attractions, tips, etc.)

- 💬 Gradio chatbot UI


![Uploading Screenshot 2025-12-01 at 5.58.40 PM.png…]()

### 🧱 1. Technical Architecture
The system combines classical web development (Flask) with modern AI components (LangChain, Pinecone, LLMs). The idea is simple: ingest a PDF, convert it into searchable vectors, retrieve the most relevant text for each query, and generate a clean answer using an LLM.
- The system starts with a curated Travel PDF dataset, which is loaded and cleaned to extract raw text.
- A text splitter divides the PDF text into small, meaningful chunks that preserve context and improve retrieval accuracy.
- These chunks are transformed into dense numerical vectors using HuggingFace Sentence Transformer embeddings, enabling semantic understanding.
- The resulting vectors are stored inside a Pinecone vector database, which allows fast similarity search based on meaning rather than keywords.
- When a user asks a question, LangChain’s retriever fetches the top-K most relevant chunks from Pinecone.
- These retrieved chunks are then inserted into the LLM prompt, ensuring the model only uses verified context when generating an answer.
- GPT-4o processes this enriched prompt and produces a grounded, context-aware final answer with no hallucination.
- The Flask frontend handles the conversation, sending user queries into the RAG pipeline and displaying the AI-generated responses in real time.
- Gradio serves as the user-facing chat interface, turning the backend RAG pipeline into an interactive web application.

#### 📘 Step 1 — Loading and Processing the Travel PDF
-The first major step in the system involves loading the travel PDF and preparing it for embedding. LangChain provides utility classes that simplify this process. I used DirectoryLoader and PyPDFLoader to read every page of the PDF and convert it into LangChain Document objects.
- This step scans the specified directory and loads every PDF file inside it. Each page of each PDF becomes a separate Document object containing page_content and metadata. The reason for loading page-by-page is that smaller text units yield better semantic search performance later.
- After loading, the documents often contain unnecessary metadata such as page numbers, file paths, and extraction artifacts. To keep the index clean, I reduced metadata to only the “source” value.
- The next step is chunking the text. Long paragraphs cannot be embedded effectively as single units, so they must be split into smaller chunks.


#### 📘 Step 2 — Creating Embeddings Using HuggingFace MiniLM
Once the text is chunked, each chunk must be converted into a numerical vector representation. These vectors represent semantic meaning and enable similarity search in Pinecone. I used the MiniLM-L6-v2 embedding model because it provides a good balance of speed and accuracy.


####  📘 Step 3 — Storing the Embeddings in Pinecone
- After generating embeddings for all chunks, they need to be stored in a vector database. Pinecone was chosen because it provides fast, scalable similarity search optimized for production systems. I created a Pinecone index and connected to it.The index already contains embedded travel PDF chunks. Pinecone is responsible for comparing the vector representation of user queries to all stored vectors and returning the closest matches.
- To retrieve relevant text later, I wrapped the vector store inside a LangChain retriever. 
- The retriever returns the top three most relevant chunks for each user prompt. Choosing a value of three strikes a balance between relevance and token constraints. Retrieving too many chunks reduces precision and increases the chance of polluting the prompt.

#### 📘 Step 4 — Setting Up GPT-4o as the LLM Layer
- The Retrieval-Augmented Generation pipeline uses GPT-4o-mini for the final answer generation. The model is initialized with a temperature of zero to ensure deterministic and factual responses.
- A system prompt is used to enforce strict ground rules. The model is instructed to provide concise travel-related answers using only the retrieved context and not rely on external knowledge. This prevents hallucination.

#### 📘 Step 5 — Building the Retrieval-Augmented Generation (RAG) Chain
- LangChain offers prebuilt utilities to combine the retriever and the generative model. First, I built a document-stuffing chain that allows the model to incorporate retrieved documents into its prompt.
- The first chain embeds the retrieved chunks into the prompt template. The second chain invokes retrieval and then generation in a single unified call. When a query arrives, the system retrieves relevant travel information, injects it into the prompt, passes it to GPT-4o-mini, and returns the final answer.

#### 📘 Step 6 — Integrating the Entire RAG System with Flask
After designing the retrieval pipeline, I connected everything to a Flask application to serve the chatbot through a web interface.

#### 🧩 Step 7- Adding a User-Friendly Chat Interface Using Gradio
- After building the full Retrieval-Augmented Generation (RAG) pipeline with Flask, LangChain, Pinecone, HuggingFace embeddings, and GPT-4o-mini, the next step was making the system easy and intuitive for users. To accomplish this, I added a Gradio-based chat interface that allows anyone to interact with the chatbot without writing any code.
- This interface allows users to simply type a question - such as "What are the best attractions in Toronto?" - and see the AI respond instantly with a grounded, factual answer retrieved from the PDF knowledge base.


### 🧩 The entire architecture works by combining document ingestion, semantic embedding, vector storage, retrieval, and controlled generation.
- HuggingFace Embeddings are responsible for converting all text chunks extracted from the PDF into dense numerical vectors that capture semantic meaning.
- Pinecone acts as the vector database and performs fast similarity search across those embeddings to find the most relevant pieces of information for any user query.
- The LangChain Retriever sits on top of Pinecone and orchestrates the Top-K retrieval process, ensuring that only the most useful context chunks are returned to the model.
- GPT-4o then uses this retrieved context to generate the final answer while staying grounded in the provided information. Flask serves as the lightweight web server and API layer that exposes the entire pipeline to a chat interface.
- The full RAG pipeline, combining retrieval and generation, ensures that the model remains factual and avoids hallucinations by always basing its responses on retrieved PDF knowledge.

The pipeline transforms an unstructured travel PDF into an intelligent, conversational assistant that always answers with factual information and avoids hallucinations.


