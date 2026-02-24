# Multi-Agents RAG System

## 📖 Overview
This project implements an Enterprise-grade Retrieval-Augmented Generation (RAG) system. The pipeline is designed to transform raw PDF documents into searchable knowledge and interact with it through a highly optimized conversational interface.

## 🏗️ Architecture

As outlined in `architecture.md` and implemented in `RagChatbot.ipynb`, the system is divided into two main components:

### Phase 1: The Ingestion Loop (Data Preparation)
- **Data Connector**: Loads PDF documents from the local `data/` directory using `PyPDFLoader`.
- **Document Cleaning**: Cleans noise such as page numbers, repetitive footers, and excessive whitespaces using Regex.
- **Chunking**: Splits documentation into smaller nodes using `RecursiveCharacterTextSplitter` with chunks of 700 tokens and a 10% overlap (70 tokens) to ensure context continuity.
- **Embedding**: Converts text chunks into high-dimensional vectors (3072 dimensions) using OpenAI's `text-embedding-3-large`.
- **Vector Storage**: Pushes and stores the embeddings in a serverless Pinecone index (`multi-agents`). Duplicate prevention is handled via `md5` hashing on chunk content.

### Phase 2: The Inference Loop (The Chatbot)
- **Query Transformation**: Leverages `gpt-4o` with a `0.0` temperature to rewrite user queries into specific, search-friendly formats, resolving acronyms and ambiguous language.
- **Retrieval (Hybrid Search)**: Fetches the top 20 candidate document chunks from Pinecone.
- **Reranking**: Employs an advanced Cross-Encoder via Cohere (`rerank-v3.5`) to meticulously re-score the top 20 candidates and select the absolute **top 5 most relevant documents**.
- **Context Assembly & Generation**: Bundles the 5 best documents with source citations and uses `gpt-4o` (at a very low temperature of `0.1`) to ensure fully factual generation strictly bound to the provided context.

## 🛠️ Prerequisites
- Python 3.9+
- [OpenAI API Key](https://platform.openai.com/api-keys)
- [Pinecone API Key](https://app.pinecone.io/)
- [Cohere API Key](https://dashboard.cohere.com/api-keys)

## 🚀 Setup & Installation
1. Clone the repository and navigate to the project root.
2. Install the required dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Create a `.env` file in the root directory and securely add your API keys:
   ```env
   OPENAI_API_KEY="your_openai_key"
   PINECONE_API_KEY="your_pinecone_key"
   COHERE_API_KEY="your_cohere_key"
   ```
4. Place your PDF documents inside the `data/` folder.  
   *(Note: The actual PDF files in this folder are ignored by Git to keep the repository lightweight and your proprietary data secure.)*

## 💻 Usage
The primary execution environment is currently the Jupyter Notebook `RagChatbot.ipynb`.

1. **Run Phase 1 (Ingestion)**: 
   Open the notebook and execute the cells under **Phase 1: The Ingestion Loop**. This process will load your PDFs, clean them, chunk them, and upload the embeddings to your configured Pinecone index.
   
2. **Run Phase 2 (Inference)**: 
   Execute the remaining cells under **Phase 2: The Inference Loop**. You can then use the `ask(question: str)` function directly within the notebook to query your documents.
   
   **Example:**
   ```python
   ask("According to the document what is the current version and who is the author of locindoor")
   ```

## 🧰 Tech Stack
- **Frameworks**: LangChain, LangChain Core/Community
- **LLM & Embeddings**: OpenAI (`gpt-4o`, `text-embedding-3-large`)
- **Vector Database**: Pinecone
- **Reranker**: Cohere (`rerank-v3.5`)
- **Document Loaders**: PyPDF, Unstructured
