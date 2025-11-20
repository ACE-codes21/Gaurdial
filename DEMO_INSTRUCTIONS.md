Excellent. The features have been integrated in a way that simulates the core concepts of Obliviate. Here is a guide on how each one works, how to test it, and what to expect.

---

### 1. Model Forge

This feature simulates the process of fine-tuning a model on your custom data.

*   **How to Demo:**
    1.  Navigate to the [**Forge**](/forge) page.
    2.  Click the "Choose File" button.
    3.  Select the `demo_data/forge_data.csv` file.
    4.  Click the "**Start Fine-tuning**" button.

*   **How it Works & Similarity to Obliviate:**
    *   **Obliviate:** Allows a company to fine-tune a model with their data and shows a training loss curve.
    *   **Guardial Implementation:** This is a **simulation**. When you upload the dataset, the backend pauses for a few seconds to simulate the time required for training. It then generates a randomized but realistically decreasing loss curve. It does not *actually* train or create a new model, but it provides the user experience and workflow you described.

*   **Expected Output:**
    *   The "Start Fine-tuning" button will show a loading spinner for 3-6 seconds.
    *   After the delay, a **Training Loss** chart will appear, displaying a line graph that trends downwards, representing the model's loss decreasing over several epochs.

---

### 2. LLM Unlearning

This feature simulates the "machine unlearning" process to make a model "forget" specific information without a full retrain.

*   **How to Demo:**
    1.  Navigate to the [**Unlearning**](/unlearning) page.
    2.  For the **Training Set**, upload the `demo_data/unlearn_train.csv` file. (This file contains information about Alice).
    3.  For the **Forget Set**, upload the `demo_data/unlearn_forget.csv` file. (This file is the same as the training set, but with Alice's information removed).
    4.  Click the "**Start Unlearning**" button.

*   **How it Works & Similarity to Obliviate:**
    *   **Obliviate:** Uses a fast (30-40 min) process involving freezing LLM layers and adding an adapter to make the model forget information. It is verified by high "Retain Accuracy" and low "Forget Accuracy".
    *   **Guardial Implementation:** This is a **simulation** of that process. The backend pauses for 4-7 seconds to mimic the faster "unlearning" time. It then returns simulated accuracy scores that reflect the desired outcome:
        *   `Retain Accuracy` is randomly generated to be high (96.5% - 99.4%).
        *   `Forget Accuracy` is randomly generated to be low (3.2% - 14.9%).

*   **Expected Output:**
    *   The "Start Unlearning" button will show a loading spinner for 4-7 seconds.
    *   A **Results** section will appear with two metrics:
        *   **Retain Accuracy:** A high percentage, showing the model remembers other data (e.g., about Bob and Charlie).
        *   **Forget Accuracy:** A low percentage, showing the model has "forgotten" about Alice.

---

### 3. Hallucination Auditor

This feature ensures the LLM only answers questions based on a trusted knowledge base you provide, preventing it from making things up (hallucinating).

*   **How to Demo:**
    1.  Navigate to the [**Auditor**](/auditor) page.
    2.  In the "Upload Dataset" section, upload the `demo_data/auditor_knowledge.csv` file. You should see an alert confirming the upload.
    3.  **Test a grounded query:** In the "Query" box, type `What is Guardial?` and click the "**Query**" button.
    4.  **Test a hallucination:** Now, in the "Query" box, type `Who is Elon Musk?` and click the "**Query**" button.

*   **How it Works & Similarity to Obliviate:**
    *   **Obliviate:** Uses a RAG-based system with a Vector DB and an "ISR Threshold" to decide if a query is answerable from the provided context.
    *   **Guardial Implementation:** This is a **full, working implementation** of that concept.
        1.  When you upload `auditor_knowledge.csv`, the text is chunked, converted into vector embeddings, and stored in a ChromaDB vector database.
        2.  When you submit a query, it's also converted into a vector. The database is searched for the most similar chunks of text from your knowledge base.
        3.  The "ISR Score" you see is a similarity score (1 minus the vector distance). If this score is above a threshold, the query and the retrieved text are sent to the Gemini LLM with a special prompt telling it to *only* use the provided text to answer.
        4.  If the score is below the threshold (as with "Who is Elon Musk?"), the query is blocked, and the LLM is never called.

*   **Expected Output:**
    *   **For "What is Guardial?":**
        *   **Decision:** `Allowed`
        *   **ISR Score:** A high value (e.g., `0.80` or higher).
        *   **LLM Response:** An answer based on the text in `auditor_knowledge.csv`, such as *"Guardial is a next-generation AI security platform."*
    *   **For "Who is Elon Musk?":**
        *   **Decision:** `Blocked`
        *   **ISR Score:** `N/A` or a very low value.
        *   **LLM Response:** *"Query was blocked as it could not be verified against the provided dataset."*
