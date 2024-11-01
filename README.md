# LangGraph-AstraDB-Project

The code creates a retrieval-based AI system using several language model and retrieval libraries, particularly LangChain, Cassio, and LangGraph. Below is a breakdown of the code:

### 1. Installation of Libraries

The first few lines install necessary libraries for handling language models, embeddings, and vector storage. Packages such as langchain, langgraph, cassio, langchain_community, and chromadb are key tools for document retrieval, text splitting, and vector search.

### 2. Astra DB Connection

```python
import cassio
ASTRA_DB_APPLICATION_TOKEN = "..."
ASTRA_DB_ID = "..."
cassio.init(token=ASTRA_DB_APPLICATION_TOKEN, database_id=ASTRA_DB_ID)
```

This connects to Astra DB (a cloud-based database by DataStax) using a secure token and ID. This allows storage and retrieval of document vectors.

### 3. Document Loading, Splitting, and Indexing

- Loading Documents: Using URLs, the code loads documents through the WebBaseLoader.

- Splitting Text: Texts are split into manageable chunks (500 characters, no overlap) for indexing, improving search efficiency.

- Embedding Creation: The HuggingFaceEmbeddings model ("all-MiniLM-L6-v2") is used to convert text into vector embeddings.

- Storing Embeddings in Cassandra: The Cassandra vector store is used to store the generated embeddings. This allows fast retrieval of semantically similar documents.

### 4. Vector Store Indexing and Retrieval

```python
retriever = astra_vector_store.as_retriever()
retriever.invoke("What is agent", ConsistencyLevel="LOCAL_ONE")
```

This creates a retriever for querying the stored embeddings. When queried (e.g., "What is agent"), it will find the most relevant documents from the stored vectors.

### 5. Routing System

- Routing Model: The RouteQuery class is defined to decide if the question should be answered by the vector store or Wikipedia.
- Prompt Template: The system prompt is set to route questions to Wikipedia if they're not related to "agents", "prompt engineering", or "adversarial attacks".
- LLM-Based Router: Using ChatGroq (a language model), a prompt router decides the data source for each query.

### 6. Setting Up Wikipedia and Arxiv Search

ArxivAPIWrapper and WikipediaAPIWrapper are tools to query academic articles and Wikipedia articles, respectively, allowing the code to access broader information.

### 7. Graph State Management

```python
class GraphState(TypedDict):
    question: str
    generation: str
    documents: List[str]
```

This defines the graph state for storing question data, generated responses, and document lists.

### 8. Functions for Document Retrieval and Wiki Search

- retrieve: Retrieves documents using the vector store.

- wiki_search: Retrieves documents using Wikipedia.

### 9. Conditional Routing

```python
def route_question(state):
    source = question_router.invoke({"question": question})
    if source.datasource == "wiki_search":
        return "wiki_search"
    elif source.datasource == "vectorstore":
        return "vectorstore"
```

This function routes a question to either Wikipedia or the vector store based on its content.

### 10. Graph Workflow

```python
workflow = StateGraph(GraphState)
```

This creates a graph workflow that conditionally calls either the wiki_search or retrieve nodes based on the route determined by route_question.

### 11. Running the Workflow
The workflow is executed with example questions, routing and fetching the relevant answers for each. It prints out the graph's decisions, outputting either Wikipedia articles or vector store results.

### 12. Visualization and Final Outputs
The code displays the graph with Mermaid and prints the final documents, depending on whether the search was routed to Wikipedia or the vector store.
