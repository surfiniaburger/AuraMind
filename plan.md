# Plan for Implementing Retrieval Augmented Generation (RAG) in Gallery Android App

**Goal:** Implement an on-device (or hybrid) RAG system for the Android "gallery" app to answer user queries by retrieving relevant information from a local knowledge base and using it to inform a generative model's response.

## Core Components Recap:
1.  **Knowledge Base Storage (On-Device):** Using Room.
2.  **Text Embedding Generation (On-Device):** Using a TFLite sentence embedding model.
3.  **Vector Index & Search (On-Device):** For efficient similarity search of embeddings.
4.  **Ingestion Pipeline:** To process and store knowledge.
5.  **Retrieval Pipeline:** To fetch relevant context for a query.
6.  **Generative Model Integration:** To produce the final answer.
7.  **Security:** To protect stored data.

## Implementation Steps & Milestones:

---

### Milestone 1: Foundational Data Storage with Room

*   **Step 1.1: Add Room Dependencies.**
    *   Verify or add Room dependencies (`room-runtime`, `room-compiler`, `room-ktx`) to your `build.gradle` file.
*   **Step 1.2: Define Data Entities.**
    *   Create Kotlin data classes for `Document` (e.g., `id`, `name`, `source_description`) and `TextChunk` (e.g., `id`, `documentId`, `chunk_text`, `page_number`). Annotate them as Room entities.
*   **Step 1.3: Create Data Access Objects (DAOs).**
    *   Define Room DAOs with methods to insert, query, and delete `Document` and `TextChunk` records.
*   **Step 1.4: Setup Room Database.**
    *   Create a class that extends `RoomDatabase`.
    *   Implement a singleton pattern for accessing the database instance.
*   **Step 1.5: Basic Testing.**
    *   Write simple unit tests or in-app test code to verify that you can insert and retrieve data from the Room database.

---

### Milestone 2: On-Device Text Embedding Capability

*   **Step 2.1: Select and Acquire TFLite Embedding Model.**
    *   Research and download a suitable TFLite sentence embedding model (e.g., Universal Sentence Encoder Lite). Consider model size, performance, and embedding dimensions.
*   **Step 2.2: Add Model to Project.**
    *   Place the `.tflite` model file in your app's `assets` folder.
*   **Step 2.3: Add TensorFlow Lite Dependencies.**
    *   Add the `tensorflow-lite-support` and `tensorflow-lite-task-text` (if applicable for easier API) or core `tensorflow-lite` dependencies to `build.gradle`.
*   **Step 2.4: Implement Embedding Service.**
    *   Create a class/service to:
        *   Load the TFLite model from assets.
        *   Preprocess input text strings into the format expected by the model.
        *   Run inference to generate text embeddings (vector representations).
        *   Post-process the output if necessary.
*   **Step 2.5: Basic Testing.**
    *   Test the embedding service with sample text inputs and verify that it produces numerical vectors without errors.

---

### Milestone 3: Initial Data Ingestion Pipeline

*   **Step 3.1: Prepare Sample Knowledge Data.**
    *   Have a few sample text files (e.g., similar to `healthy_maize_remedy.txt`, `maize_phosphorus_deficiency_remedy.txt`) ready.
*   **Step 3.2: Develop Ingestion Logic.**
    *   Create a mechanism (e.g., a function, a WorkManager task) that:
        *   Reads text from your sample data sources (e.g., files in assets).
        *   Optionally, splits the text into smaller, manageable chunks.
        *   For each chunk:
            *   Uses the `EmbeddingService` (Milestone 2) to generate its embedding.
            *   Saves the original text chunk and its associated document info to the Room database (Milestone 1).
            *   **(Placeholder for Milestone 4):** Note where you *would* store the embedding itself for indexing. Initially, you might store it directly in Room alongside the chunk if your vector search library allows, or prepare for an external index.
*   **Step 3.3: Trigger Ingestion.**
    *   Implement a way to trigger this ingestion (e.g., on first app launch, or a manual button for testing).
*   **Step 3.4: Verification.**
    *   Verify that text chunks are stored in Room.
    *   Verify that embeddings are being generated (you can log them or inspect them during debugging).

---

### Milestone 4: Vector Indexing & Search (Proof of Concept)

*   **Step 4.1: Research and Choose On-Device Vector Search Method.**
    *   Evaluate options:
        *   Lightweight k-NN Java/Kotlin libraries.
        *   Storing embeddings in Room and performing cosine similarity searches (feasible for very small datasets or initial PoC).
        *   Potentially, investigate compiling a lightweight version of FAISS or ScaNN for Android (more complex).
*   **Step 4.2: Integrate Chosen Solution.**
    *   Add any necessary dependencies for the chosen library.
*   **Step 4.3: Index Existing Embeddings.**
    *   Modify the ingestion pipeline (Milestone 3) or create a separate process to take the generated text chunk embeddings and add them to your chosen vector index. The index should store a reference back to the `TextChunk` ID in Room.
*   **Step 4.4: Implement Basic Search Function.**
    *   Create a function that:
        *   Takes a user query (text).
        *   Generates an embedding for the query using the `EmbeddingService`.
        *   Searches the vector index with the query embedding to find the top-K most similar chunk embeddings.
        *   Retrieves the IDs of these chunks.
*   **Step 4.5: Testing.**
    *   Test the search with sample queries and verify it returns relevant chunk IDs.

---

### Milestone 5: Full Retrieval Pipeline

*   **Step 5.1: Connect Search to Data Retrieval.**
    *   Enhance the search function (Milestone 4.4) to use the retrieved chunk IDs to fetch the full text of those chunks from the