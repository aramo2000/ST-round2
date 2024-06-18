**RAG (Retrieval augmented generation)** is a generative AI method that enhances LLM performance by combining world knowledge with custom or private knowledge. This combining of knowledge sets in RAG is helpful for several reasons: 
* Providing LLMs with certain information regarding certain fields: In our case, we want the LLMs to work and generate responses specifically from installation manual PDFs. RAG allows adding new knowledge without retraining a whole LLM model from scratch.
* Maintaining a dynamic knowledge base: Custom documents can be updated, added, removed, or modified anytime, keeping RAG systems up-to-date without retraining.
 <br>
 
**1- Describe each component of the RAG system (embedder, PDF processor, vector DB, LLM response generation etc.). For each component, describe how you would choose which tool to use (e.g. for the embedding model I will choose to use openAI embeddings because of ….). You can also present certain tools and discuss why you wouldn't use them.**

**PDF Processor.**<br>
This process handles the extraction of text and metadata from the given PDF files to ensure the data in those PDFs are ready to be indexed and queried. There are certain steps involved in processing the PDFs:<br>
* Extracting Text from PDFs: The first step is to read the content from the PDF files and convert it into plain text.<br>
* Segmenting the Text: After extracting the text, the next step is to break it down into smaller, more manageable chunks. This helps in efficient indexing and retrieval.<br>
* Extracting Metadata: Metadata such as section titles, headings, and subheadings can provide valuable context for better query responses. We can enhance the text extraction process to capture this metadata.<br>
* Preprocessing Text: Additional preprocessing steps can improve the quality of the text data, such as removing unnecessary whitespace, normalizing characters, or handling special cases.<br>

Tool to achieve this: **PyMuPDF** <br>
Here's How PyMuPDF Can Deliver for RAG: <br>
* **Data Extraction:** PyMuPDF allows us to extract text, tables, images and vector graphics from documents accurately and in a context-preserving way. This functionality is crucial for the retriever module in RAG, as it enables the system to access the content of PDF documents and identify relevant passages based on the input query.<br>
* **Document Processing:** PyMuPDF provides features for processing PDF documents, such as splitting, merging, and manipulating pages. This can be useful for preprocessing documents before retrieval, such as splitting large documents into smaller sections or removing irrelevant pages.<br>
* **Indexing:** PyMuPDF can assist in creating indexes or databases of document content. By extracting text and organizing it in a structured format, PyMuPDF enables efficient searching and retrieval of information during the retrieval phase of RAG.<br>
* **Efficiency:** PyMuPDF is known for its efficiency. It is designed to be fast and lightweight, making it suitable for processing large volumes of documents efficiently. This efficiency is essential in the RAG framework, where the retriever module needs to quickly scan through a large corpus of documents to find relevant passages.<br>

There's another tool that I am not considering to use is **PDFMiner**. Even though it has very detailed text extraction capabilities, it can be slower, especially for larger PDFs. It tends to be more resource-intensive due to its detailed text extraction capabilities. Also, it will use more memory during the extraction phase. This will be better suited for applications needing detailed structural extraction rather than straightforward text extraction.
<br>
<br>
**Embedder.**<br>
In the simplest approach to RAG, the application will use the user's query to perform a semantic search to retrieve relevant search results. In order to perform this semantic search, the RAG application will use an embedding model. The embedding model will take the text from the user's prompt and it will produce an output vector called an embedding (it will encode the text). The text will be projected into a numerical vector for efficient similarity search and retrieval, which will be transferred to the vector database (which will be explained later). The Processed PDF data will also be embedded and stored in a vector database for later comparisons.

Tool to achieve this: **OpenAI's text-embedding-3-small model.** <br>
Importance of this model for our RAG:<br>
* **High-Quality Embeddings:** The model generates high-dimensional embeddings (1536 dimensions) that capture semantic meanings effectively. This high-dimensional space allows for nuanced differentiation between texts, which is crucial for accurate information retrieval.<br>
* **Improved Semantic Understanding:** Compared to its predecessors text-embedding-ada-002, the embedding-3-small model has improved semantic understanding, allowing for better matching of queries with relevant documents. This results in higher accuracy and relevance in the retrieved information (5 times more pages for the same price, with 1.3% more accuracy).<br>
* **Efficency:** Despite its high-dimensional output, the model is designed to be efficient in computation, price, and storage compared with the old models, making it practical for large-scale applications where performance is critical.<br>
* **Integration with RAG:** When integrated into RAG systems, the model enhances the retrieval component by ensuring that the retrieved documents or passages are highly relevant to the input query. This leads to better performance in downstream generation tasks, as the generative model has access to more pertinent information.<br>

Tools not considering: <br>
* OpenAI's text-embedding-ada-002, because the new OpenAI model is a more powerful embedder and five times less expensive<br>
* OpenAI's text-embedding-3-large model offers 2.3% better performance than the 3-small model, but its price is around six times more than the 3-small model.<br>
<br>

**Vector Database.** <br>
The vector database is a specialized database designed to store and manage the embeddings generated from the embedding model. The vector database retrieves relevant information based on the similarity of the query with the pdf data vectors.<br>

To store the vector DB, I'll use **Weaviate:** <br>
* **Optimized for Vectors:** It is specifically designed for handling high-dimensional vector data efficiently. As we are using an embeddor that generates very high-dimensional embeddings, Weaviate will be perfect for us.<br>
* **Content Management Systems:** In PDFs, there could be many knowledge graphs or unstructured data. Weaviate offers flexibility in data structuring and retrieval capabilities, featuring a schema-driven approach.<br>
* **Complex Search and Recommendation Systems:** As we could receive some complex queries, weaviate's vector similarity searches with traditional text-based searches can leverage the search effectively.<br>

Even though **Pinecone DB** offers better scalability, for starters, we can leverage Weaviate's management system and vector optimization, and later, if we need high scaling, we can move the organized data to Pinecone (which I guess we won't need as we won't receive, at least during the early stages, very high request through our RAG)
<br>
<br>

**LLM Response Generator.** <br>
Once the relevant information has been found in the previous step, the retrieved data is used as input to a Large Language Model (LLM). Here's how the response generation step unfolds:
* Each retrieved document or passage is provided as input to the LLM. This input is typically tokenized and encoded into a numerical format suitable for the model. <br>
* The LLM integrates the retrieved information as context. This means that the model considers the retrieved texts alongside the original query or prompt to generate a response that is coherent and relevant. <br>
* The LLM then generates a response based on the combined context of the original query/prompt and the retrieved information. <br>
We can use OpenAI's GPT-4-Turbo as the response generator as it is mainly the latest model with advanced capabilities in generating high-quality, contextually relevant responses through the given data. It builds upon the capabilities of GPT-3, providing better contextual understanding and more accurate responses to a wide range of queries.
<br>

**Prompt Engineer and Data retrieval.** <br>
It is very important to use small **prompt engineering** so that the model knows that it is an assistant bot, and the users will ask questions related to a certain PDF manual connected with equipment. Then, we can use **GPT 4's retrieval tool** and make it answer based only on the information it receives from the vector DB.
 <br>
 <br>
  <br>
**2- Describe at least 2 challenges that you will encounter when using the tool(s) that you chose (e.g. when using openAI embeddings, the vector size is too long …), and try to think of how you can overcome those challenges.**

1- A challenge could be that I explained above is the vector database's scalability. If this AI is used by too many technicians, we will need to scale our database, and while Weaviate is designed for scalability, there are some practical limits to how much it can scale in certain environments or configurations. These limits can depend on factors such as hardware capabilities, network infrastructure, and the complexity of data and queries. If we see that we are about to reach to those limits, we will need to upgrade our whole vector DB to a more scalable one, for example, Pinecone, even though that will not offer a better management system.

2- We may require to Fine-Tune our GPT-4-Turbo generator model so that the model can adapt to the specific dataset relevant to our RAG. This adaptation ensures that the generated responses are more contextually appropriate and aligned with the target use case. Also, this helps improve the accuracy of the generated responses by reducing generalization errors and aligning the model's language generation capabilities with the specific nuances of the answer to the query. This is considered a challenge that we may be required to handle after releasing the AI system.

3- There also could be a challenge in a delay during the embedding process using OpenAI's text-embedding-3-small model. In some cases, it is helpful to cache frequently accessed embeddings and pre-fetch embeddings for anticipated queries. This can significantly reduce latency for common generalized queries (that may be the same for different manuals).
 <br>
  <br>
  <br>
**3- Once you complete choosing the tools, present 5 examples of complex questions that the chatbot you designed will be able to answer, and 5 examples of questions that your chatbot will fail to answer. Present reasons why.**

Will be able to answer:

**How do I reset the filter indicator light on my Samsung refrigerator?** <br>
Even though there could be many Samsung refrigerators, the AI will use data from different manuals to generate a generalized response for the question, although the answer could not be correct, and the user may need to provide the model number/name to get a more accurate response.

**What factors should I consider when choosing between a tankless and a traditional tank water heater for a large household?** <br>
The chatbot can discuss the benefits of tankless water heaters in terms of energy efficiency and continuous hot water supply versus the higher initial cost and installation complexity compared to traditional tank heaters.

**Can I install the Bosch tankless water heater in an outdoor location?** <br>
The chatbot can search for installation requirements through the manual and provide instructions about outdoor vs indoor capabilities if the details are present in the manual.

**How do I troubleshoot a blinking error code on my Panasonic microwave 345py?** <br>
The chatbot can search data connected with blinking error codes and troubleshooting and provide the featured steps, if available.

**What steps should I follow to fix a leaky kitchen faucet?** <br>
The chatbot will be able to answer the question even without having any data connected with the kitchen faucet because it will have the general steps to the query, although it may not be applicable to every faucet.
<br>

<br>
Will not be able to answer:

**How can I create a RAG chatbot system?** <br>
As we are using GPT4's retrieval tool, the AI will only try to answer the question through the data available in the vector DB, and this data will not be there (as it is limited to the content of the provided manuals), so it will fail to answer the question (even though GPT-4-Turbo is capable of answering this)

**Will the Samsung refrigerator fit in my kitchen free space?** <br>
The AI will need data connected to the kitchen's free space and will need to know which Samsung refrigerator is being used, so it will fail to answer without any additional context.

**What is the best brand of water heater on the market?** <br>
This is an opinion-based question requiring market analysis and personal preferences, which the chatbot is not equipped to handle. It lacks the ability to provide subjective opinions or compare brands beyond the scope of the given manuals.

**Can you diagnose the unusual noise my HVAC system is making?** <br>
Diagnosing specific noises or sounds typically requires physical inspection and subjective analysis that the chatbot cannot perform. The chatbot will only be able to offer general troubleshooting advice from the manual, not specific diagnostics based on audio/image/video descriptions.

**What roofing material would you recommend for a coastal home prone to high humidity, salt exposure, and frequent storms, considering durability, maintenance requirements, and aesthetic appeal?** <br>
This question involves considerations beyond basic installation instructions, such as regional climate factors, environmental exposure, and specific aesthetic preferences, which require expert judgment and local knowledge.

<br>
<br>

Some used resources along with chatGPT: <br>
https://www.willowtreeapps.com/craft/retrieval-augmented-generation#:~:text=Key%20Components%20of%20a%20RAG,LLM%20and%20its%20associated%20prompt
https://medium.com/@pymupdf/rag-llm-and-pdf-enhanced-text-extraction-5c5194c3885c
https://vectorize.io/picking-the-best-embedding-model-for-rag/#:~:text=In%20order%20to%20perform%20this,query%20to%20the%20vector%20database.
https://platform.openai.com/docs/guides/embeddings/what-are-embeddings
https://medium.com/intel-tech/optimize-vector-databases-enhance-rag-driven-generative-ai-90c10416cb9c#:~:text=Some%20examples%20of%20traditional%20databases,Qdrant%2C%20Faiss%2C%20and%20Chroma.
https://platform.openai.com/docs/models
