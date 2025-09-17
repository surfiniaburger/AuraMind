### **Summary of Key Changes:**

*   **Embedding Model:** Switched from a generic model to the specific `EmbeddingGemma` model mentioned in the documentation.
*   **Inference Engine:** Replaced the general TensorFlow Lite dependencies with the `LiteRT API`, which is explicitly used in the documentation for on-device inference and includes GPU support.
*   **Tokenization:** Added a crucial, separate step for tokenization using the Deep Java Library (DJL), as the LiteRT API expects token IDs as input, not raw text.
*   **Prompting:** Incorporated the critical step of adding a specific prefix (e.g., `"task: search result | query: "`) to the text before embedding it, which significantly improves search performance.
*   **Vector Search:** For the initial proof-of-concept, the plan now suggests using the exact `cosineSimilarity` function from the documentation, which is a simple and effective starting point.

---

### **Revised Plan for Implementing RAG with EmbeddingGemma**

**Goal:** Implement an on-device (or hybrid) RAG system for the Android "gallery" app to answer user queries by retrieving relevant information from a local knowledge base and using it to inform a generative model's response.

#### **Milestone 1: Foundational Data Storage with Room**

*(This milestone remains unchanged. Your current plan is solid.)*

*   **Step 1.1: Add Room Dependencies.**
*   **Step 1.2: Define Data Entities.**
*   **Step 1.3: Create Data Access Objects (DAOs).**
*   **Step 1.4: Setup Room Database.**
*   **Step 1.5: Basic Testing.**

---

#### **Milestone 2: On-Device Text Embedding with EmbeddingGemma**

*   **Step 2.1: Select and Acquire Model & Tokenizer.**
    *   Download the **`EmbeddingGemma` TFLite model** (e.g., `embeddinggemma-300m.tflite`) from the repository mentioned in the documentation.
    *   Download the corresponding **HuggingFace tokenizer file** (e.g., `tokenizer_embedding_300m.json`).
*   **Step 2.2: Add Assets to Project.**
    *   Place the `.tflite` model file and the `tokenizer.json` file in your app's `assets` folder.
*   **Step 2.3: Add LiteRT and DJL Dependencies.**
    *   As per the documentation, add the **LiteRT** dependencies to your `build.gradle` to handle model inference and GPU delegation.
        ```groovy
        implementation("com.google.ai.edge.litert:litert:1.4.0")
        implementation("com.google.ai.edge.litert:litert-support:1.4.0")
        implementation("com.google.ai.edge.litert:litert-gpu:1.4.0")
        implementation("com.google.ai.edge.litert:litert-gpu-api:1.4.0")
        ```
    *   Add the **Deep Java Library (DJL)** dependency to handle the tokenization process.
*   **Step 2.4: Implement Embedding Service.**
    *   This service will now consist of three distinct parts as detailed in the documentation:
    *   **1. Tokenizer:** Create a class that uses DJL's `HuggingFaceTokenizer` to load the `tokenizer.json` from assets. This class will have a method to convert an input text string into an array of token IDs.
        ```kotlin
        val tokenizer = HuggingFaceTokenizer.newInstance(Paths.get("path/to/tokenizer.json"))
        val encoding = tokenizer.encode(fullInput)
        val tokenIds = encoding.ids
        ```
    *   **2. LiteRT Interpreter:** Create a class to load the `EmbeddingGemma` model using the LiteRT `Interpreter`. Use the provided code to enable the GPU delegate for better performance.
        ```kotlin
        val options = Interpreter.Options()
        // Add GPU delegate logic from documentation
        interpreter = Interpreter(loadModelFile(), options)
        ```
    *   **3. Embedding Generation:** Create a primary function that orchestrates the process:
        *   Accepts a string of text.
        *   **Applies a prompt:** Prepends the text with an instructional prompt. For search queries, this would be `"task: search result | query: "`. For documents being indexed, you might use a different prompt like `"task: document | content: "`.
        *   Uses the tokenizer to convert the full prompted text into token IDs.
        *   Pads the token ID sequence as required by the model.
        *   Feeds the token IDs into the LiteRT `interpreter.run()` method.
        *   Returns the resulting float array (the embedding vector).
*   **Step 2.5: Basic Testing.**
    *   Test the embedding service by providing a sample sentence, ensuring it returns a numerical vector of the correct size (e.g., 768 for `EmbeddingGemma`) without errors.

---

#### **Milestone 3: Initial Data Ingestion Pipeline**

*   **Step 3.1: Prepare Sample Knowledge Data.** (No change)
*   **Step 3.2: Develop Ingestion Logic.**
    *   Create a mechanism that:
        *   Reads text from your sample data sources.
        *   Splits the text into smaller chunks.
        *   For each chunk:
            *   Uses the **new `EmbeddingService`** (Milestone 2) to generate its embedding. Remember to use a document-specific prompt before tokenization.
            *   Saves the original text chunk and its metadata to the Room database (Milestone 1).
            *   **(For Milestone 4):** Store the generated float array embedding directly in a `ByteArray` or `FloatArray` column in your Room `TextChunk` entity. This is feasible for the initial proof-of-concept.
*   **Step 3.3: Trigger Ingestion.** (No change)
*   **Step 3.4: Verification.** (No change)

---

#### **Milestone 4: Vector Indexing & Search (Proof of Concept)**

*   **Step 4.1: Implement On-Device Vector Search.**
    *   Instead of researching external libraries, directly implement the **`cosineSimilarity` function** provided in the documentation. This function is lightweight and perfect for a proof-of-concept with a small dataset.
        ```kotlin
        fun cosineSimilarity(vectorA: FloatArray, vectorB: FloatArray): Float {
            // ... implementation from the documentation
        }
        ```
*   **Step 4.2: Integrate Search with Room.**
    *   No new dependencies are needed for this approach.
*   **Step 4.3: Index Existing Embeddings.**
    *   This is already handled by storing the embeddings in Room during the ingestion pipeline (Milestone 3).
*   **Step 4.4: Implement Search Function.**
    *   Create a function that:
        *   Takes a user query string.
        *   Generates an embedding for the query using the `EmbeddingService`, making sure to use the `"task: search result | query: "` prompt.
        *   Retrieves all `TextChunk` entities from Room.
        *   Iterates through them, calculating the cosine similarity between the query embedding and each stored chunk embedding.
        *   Returns the top-K `TextChunk` objects with the highest similarity scores.
*   **Step 4.5: Testing.**
    *   Test the search with sample queries and verify that it returns the text of the most relevant chunks.

---

#### **Milestone 5: Full Retrieval Pipeline**

*   **Step 5.1: Connect Search to Data Retrieval.**
    *   The search function from **Step 4.4** already accomplishes this by finding the most similar chunks and returning their data.
    *   The next step is to take the text from these top-K chunks, format it as a single context block, and prepare it for the generative model.
