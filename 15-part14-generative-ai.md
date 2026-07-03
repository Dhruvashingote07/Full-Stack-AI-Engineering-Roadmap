# Part 14: Generative AI

> **Duration:** 14 Weeks | **Difficulty:** Advanced | **Prerequisites:** Deep Learning, Transformers, NLP

---


## Chapter 101: Introduction to Generative AI

### Introduction

> 📖 **Instructor Note:** The fundamental distinction between discriminative and generative models is: discriminative models learn P(y|x) — the boundary between classes — while generative models learn P(x) — the actual data distribution. This is why generative models can create new data while discriminative models can only classify existing data.

Generative AI represents a paradigm shift in artificial intelligence — moving from systems that classify or predict to systems that create new content. At its core, generative AI models P(data) and generates novel samples from this learned distribution.

### Why It Matters

Generative AI reshapes every industry: code generation (Copilot writes 46% of new code), content creation (DALL-E, Midjourney, Sora), scientific discovery (AlphaFold), and customer experience. Market projected to reach .3 trillion by 2032.

### Real World Analogy

Discriminative: art critic who identifies styles. Generative: artist who creates new works in that style.

### Theory

**Evolution:** VAE (2013) -> GAN (2014) -> Transformer (2017) -> GPT-1 (2018) -> GPT-3 (2020) -> Stable Diffusion (2022) -> GPT-4 (2023) -> GPT-4o (2024)

**Scaling Laws (Kaplan et al., 2022):** Test loss follows power law: L(N,D) = N^(-a) + D^(-b) + L_inf. Scale model size, data, and compute together.

**Chinchilla (Hoffmann et al., 2022):** 20 tokens per parameter for optimal compute.

**Emergent abilities:** In-context learning, instruction following, reasoning, code generation appear at sufficient scale.

### Tables

| Property | Discriminative | Generative |
|----------|---------------|------------|
| Goal | P(y|x) decision boundary | P(x) data distribution |
| Output | Class label, probability | New data samples |
| Example | BERT for sentiment | GPT-4 for text generation |

### Common Mistakes

- Assuming scaling alone solves everything
- Ignoring compute budget for data
- Not validating alignment

### Best Practices

1. Match model scale to data (Chinchilla ratio)
2. Prefer small models on more data vs large on less
3. Evaluate emergent abilities at relevant scale

### Revision Notes

- Generative learns P(X), discriminative learns P(Y|X)
- Scaling laws: loss = power law of N, D, compute
- Chinchilla: 20 tokens/parameter
- Emergent abilities appear at scale thresholds
- Alignment: RLHF, Constitutional AI, red teaming

### Interview Questions

1. What is the difference between generative and discriminative models?
2. Explain the Chinchilla scaling law and its implications for model training.
3. What are emergent abilities in LLMs and at what scale do they appear?
4. How do scaling laws guide decisions about model size vs data size?
5. What is alignment and why is it necessary in generative AI?

### Practical Exercises

1. Compare the output of a generative model (GPT) vs a discriminative model (BERT) on the same input.
2. Plot the scaling law loss curve using the Kaplan et al. equation for different model sizes.
3. Calculate the optimal data-to-parameter ratio for a 7B parameter model using the Chinchilla formula.
4. Implement a simple VAE on MNIST and compare generated samples at different latent dimensions.

---

## Chapter 102: Large Language Models — Architecture

### Introduction

LLMs are transformer-based networks with billions of parameters trained on internet-scale text. Architecture evolved from GPT-1 through GPT-4o, LLaMA family, Mistral/Mixtral, and other models.

### Theory

**GPT Family:** GPT-1 (117M, decoder-only) -> GPT-2 (1.5B, zero-shot) -> GPT-3 (175B, in-context learning) -> GPT-3.5 (RLHF) -> GPT-4 (~1.8T MoE, multimodal) -> GPT-4o (omni text/vision/audio)

**LLaMA Family:** RMSNorm, SwiGLU, RoPE, no bias terms. LLaMA 2 (GQA), LLaMA 3 (128K vocab, 15T+ tokens).

**Mistral Family:** Mistral 7B (sliding window attention, GQA). Mixtral 8x7B (MoE, top-2 routing, 46.7B total/12.9B active).

**KV Cache:** Stores previous K,V during generation. GQA reduces memory (8 KV heads vs H for MHA).

### Tables

| Model | Params | Active | Vocab | Context | Training Tokens |
|-------|--------|--------|-------|---------|-----------------|
| GPT-3 | 175B | 175B | 50K | 2K | 300B |
| LLaMA 2 70B | 70B | 70B | 32K | 4K | 2T |
| LLaMA 3 70B | 70B | 70B | 128K | 8K | 15T+ |
| Mixtral 8x7B | 46.7B | 12.9B | 32K | 32K | ~5T |

### Common Mistakes

- Using MHA for inference-heavy (KV cache dominates)
- Ignoring active vs total params in MoE
- Not accounting for tokenizer differences

> 💡 **Pro Tip:** The choice between MHA, GQA, and MQA is a memory-quality tradeoff. GQA with 8 KV heads for 32 query heads reduces KV cache by 4x with minimal quality loss, making it the default choice for production LLM inference.

### Best Practices

1. Use GQA for inference-heavy workloads
2. Prefer models trained on more tokens
3. Use quantized models for local deployment

### Revision Notes

- GPT: decoder-only, 175B+, in-context learning emergent
- LLaMA: RMSNorm, SwiGLU, RoPE, open weights
- Mistral: sliding window, MoE, efficient inference
- KV cache essential; GQA reduces memory

### Interview Questions

1. Explain the difference between MHA, GQA, and MQA attention mechanisms.
2. How does KV caching work during autoregressive generation?
3. What is the advantage of Mixture of Experts (MoE) over dense models?
4. Compare LLaMA's architectural choices (RMSNorm, SwiGLU, RoPE) with GPT's.
5. How do active parameters vs total parameters affect inference cost in MoE models?

### Practical Exercises

1. Implement KV caching for a simple transformer and measure memory savings at different sequence lengths.
2. Calculate the KV cache memory for a 70B model with MHA vs GQA at 4K context.
3. Compare inference throughput of a dense model vs an MoE model with the same total parameters.
4. Profile the attention computation time for MHA, GQA, and MQA implementations.

---

## Chapter 103: Tokenization

### Introduction

Tokenization converts text to integers. BPE (GPT), WordPiece (BERT), SentencePiece (LLaMA). Tokenizer design affects context utilization, compute cost, and multilingual performance.

### Theory

**BPE:** Iteratively merges frequent byte pairs. GPT-2: 50,257 vocab. GPT-4: cl100k_base (~100K).

**LLaMA tokenizer:** SentencePiece, 32K vocab, Unigram LM.
**LLaMA 3 tokenizer:** Tiktoken BPE, 128K vocab. Reduces token count for code/non-English by 30-50%.

**Edge cases:** Numbers (123 -> "12","3"), whitespace, CJK (character-level), emoji.

> ⚠️ **Warning:** Tokenization is a common source of silent errors. If you use the wrong tokenizer for a model, the model will produce gibberish because the token-to-ID mapping is completely different. Always verify that your tokenizer matches the model checkpoint exactly.

### Best Practices

1. Always use model's original tokenizer
2. Profile tokenization for your use case
3. 128K+ vocab for multilingual apps
4. Use tiktoken for GPT models

### Revision Notes

- BPE: merge frequent byte pairs (GPT)
- WordPiece: likelihood-based merging (BERT)
- SentencePiece: unigram LM (LLaMA)
- 128K vocab optimal for multilingual + code

### Interview Questions

1. How does BPE tokenization work step by step?
2. What is the advantage of a larger vocabulary (128K vs 32K) for multilingual models?
3. How does SentencePiece differ from BPE?
4. Why do different languages have different token-to-character ratios?
5. How does tokenizer choice affect effective context length?

### Practical Exercises

1. Train a BPE tokenizer on a multilingual corpus and compare token counts across languages.
2. Tokenize the same text with GPT-2 (50K vocab) and LLaMA 3 (128K vocab) tokenizers and compare token counts.
3. Implement the BPE merge algorithm from scratch.
4. Analyze the token efficiency of your application's text with different tokenizers.

---

## Chapter 104: Prompt Engineering

### Introduction

Designing LLM inputs for desired outputs. Zero-shot, few-shot, Chain-of-Thought, Tree-of-Thought, ReAct, structured prompting, automatic optimization.

### Theory

**CoT (Wei et al., 2022):** Step-by-step reasoning. Zero-shot CoT: "Let's think step by step". GSM8K: 18% -> 82%.

**ToT (Yao et al., 2023):** Multiple reasoning paths via tree search.

**Self-Consistency (Wang et al., 2022):** Majority vote over CoT paths.

**ReAct (Yao et al., 2022):** Thought -> Action -> Observation loop.

**Parameters:** temperature (0-2), top_k (1-100), top_p (0-1), frequency_penalty (-2-2), presence_penalty (-2-2).

> 💡 **Pro Tip:** Start with temperature=0 for deterministic tasks (classification, extraction) and increase to 0.7-1.0 for creative tasks (writing, brainstorming). For multi-turn conversations, use low temperature (0.3) to maintain coherence. Always test systematically — prompt engineering is an empirical science, not an art.

### Best Practices

1. Be specific and explicit
2. Use separators for sections
3. Prime output with first words
4. Start with low temperature (0-0.3)
5. Test systematically

### Revision Notes

- Zero-shot: task only; few-shot: k examples
- CoT: step-by-step, major math/logic gains
- ToT: tree search over reasoning paths
- Self-consistency: majority vote
- ReAct: Thought-Action-Observation

### Interview Questions

1. What is Chain-of-Thought prompting and why does it improve reasoning?
2. How does Tree-of-Thought extend beyond Chain-of-Thought?
3. Explain the ReAct pattern and how it enables agentic behavior.
4. What is the role of temperature in controlling LLM output randomness?
5. How do few-shot examples affect model output compared to zero-shot?

### Practical Exercises

1. Test zero-shot vs few-shot CoT on a set of math word problems and compare accuracy.
2. Implement the ReAct pattern (Thought-Action-Observation) for a simple task like web search.
3. Experiment with temperature values (0, 0.3, 0.7, 1.0) and observe output diversity.
4. Build a prompt template library with system prompts for different roles (coder, writer, analyst).

---

## Chapter 105: Embeddings and Vector Representations

### Introduction

Dense vectors capturing semantic meaning. Models produce 384-3072 dim vectors for search, clustering, classification, RAG.

### Theory

**Similarity metrics:** Cosine ([-1,1]), Dot product (-inf, inf), Euclidean (L2, [0, inf))

**Contextual vs static:** Word2Vec/GloVe (one per word) vs BERT/GPT (dynamic per context).

**Bi-encoders vs Cross-encoders:** Bi = independent encoding (fast, retrievable), Cross = joint encoding (accurate, per-pair).

**Matryoshka embeddings:** Nested dimensions, truncatable (1024->256 with 95% accuracy).

### Tables

| Model | Dims | MTEB |
|-------|------|------|
| text-embedding-3-large | 3072 | 64.6 |
| text-embedding-3-small | 1536 | 62.3 |
| BGE-large-en-v1.5 | 1024 | 63.7 |
| all-MiniLM-L6-v2 | 384 | 56.3 |

> 💡 **Pro Tip:** L2-normalize your embeddings before storing them. With normalized embeddings, cosine similarity equals dot product, which is faster to compute and allows you to use optimized dot-product indexes in vector databases. This small step can significantly improve retrieval performance.

### Best Practices

1. Always L2-normalize embeddings
2. Match dimension to scale
3. Bi-encoders for retrieval, cross-encoders for re-ranking

### Revision Notes

- Embeddings: dense vectors for semantics
- Cosine, dot, Euclidean for comparison
- Static (Word2Vec) vs Contextual (BERT)
- Matryoshka: nested truncatable dimensions

### Interview Questions

1. What is the difference between contextual and static embeddings?
2. When would you use a bi-encoder vs a cross-encoder?
3. How do Matryoshka embeddings enable flexible storage?
4. What similarity metric works best with L2-normalized embeddings?
5. How does embedding dimension affect retrieval accuracy and storage cost?

### Practical Exercises

1. Generate embeddings for a set of documents using text-embedding-3-small and compute pairwise similarity.
2. Compare cosine similarity vs dot product on normalized vs unnormalized embeddings.
3. Implement a bi-encoder and cross-encoder for a retrieval task and compare latency.
4. Use Matryoshka embeddings to demonstrate accuracy vs dimension tradeoffs.

---

## Chapter 106: Vector Databases

### Introduction

Specialized systems for ANN search across millions-billions of vectors in milliseconds.

### Theory

**Index types:** Flat (exact, O(n*d)), IVF (k-means clustering), HNSW (multi-layer graph, best tradeoff), PQ (product quantization, memory efficient).

**Products:** Pinecone (managed), Weaviate (hybrid), Qdrant (Rust), Milvus (distributed, GPU), Chroma (lightweight), pgvector (PostgreSQL).

### Tables

| Index | Memory (10M 768d) | Time | Recall@10 |
|-------|-------------------|------|-----------|
| Flat | 30.7 GB | 5s | 1.0 |
| HNSW | 36 GB | 2ms | 0.99 |
| IVF+PQ | 960 MB | 10ms | 0.75 |

> ⚠️ **Warning:** Default vector database parameters often don't match your data distribution. Always tune HNSW's ef_construction (indexing) and ef_search (query) parameters on your actual dataset. A well-tuned index can achieve 99% recall at 10x the speed of default settings.

### Best Practices

1. HNSW as default for <100M vectors
2. Add PQ when memory constrained
3. Consider hybrid search for exact-match domains

### Revision Notes

- ANN: 95-99.9% recall, 100-1000x faster than exact
- IVF: clustering + inverted lists
- HNSW: multi-layer graph, best tradeoff
- PQ: sub-vector quantization

### Interview Questions

1. How does HNSW achieve sub-millisecond search across millions of vectors?
2. What is product quantization and how does it reduce memory usage?
3. Compare IVF, HNSW, and Flat index types in terms of speed, memory, and accuracy.
4. When would you choose a managed vector DB like Pinecone over self-hosted solutions?
5. How does hybrid search combine vector similarity with keyword filtering?

### Practical Exercises

1. Build an HNSW index on 1M random vectors and benchmark recall vs search time.
2. Implement PQ with different sub-vector sizes and measure memory/accuracy tradeoffs.
3. Compare Flat (exact) vs HNSW (approximate) search on a real embedding dataset.
4. Set up a vector database (Chroma or Qdrant) and perform a hybrid search with metadata filtering.

---

## Chapter 107: RAG (Retrieval-Augmented Generation)

### Introduction

Retrieve -> Augment -> Generate. Addresses knowledge cutoff, hallucinations, source attribution.

### Theory

**Pipeline:** Load docs -> Parse -> Chunk -> Embed -> Store -> Retrieve -> Rerank -> Generate

**Chunking:** RecursiveCharacterTextSplitter (chunk_size=512, overlap=50). Smaller chunks = precise, larger = context.

**Retrieval:** Top-k, MMR, HyDE, Multi-query, Parent-document.

**Variants:** Advanced RAG (rewrite + rerank + hybrid), Corrective RAG (self-correction), Self-RAG (reflection), Adaptive RAG (dynamic), Agentic RAG (agent decides).

**RAGAS:** Faithfulness, Answer Relevancy, Context Precision, Context Recall.

> ✅ **Best Practice:** Always re-rank retrieved documents with a cross-encoder before passing them to the LLM. Bi-encoder retrieval is fast but noisy — the cross-encoder's joint encoding of query+document significantly improves relevance. This two-stage retrieval (bi-encoder recall + cross-encoder precision) is the standard for production RAG systems.

### Best Practices

1. Chunk 256-512 tokens for Q&A
2. Always re-rank when k>3
3. Implement query rewriting (HyDE)
4. Return source documents

### Revision Notes

- RAG = Retrieve -> Augment -> Generate
- 256-512 token chunks, 10-20% overlap
- Re-ranking: cross-encoder on top-k
- RAGAS: Faithfulness, Relevancy, Precision, Recall

### Interview Questions

1. Explain the complete RAG pipeline from document ingestion to answer generation.
2. How does chunking strategy affect retrieval quality?
3. What is HyDE and how does it improve retrieval?
4. How do you evaluate RAG system quality using RAGAS metrics?
5. What is the difference between naive RAG and advanced RAG with query rewriting?

### Practical Exercises

1. Build a complete RAG pipeline using a vector database and an LLM.
2. Compare chunk sizes (128, 256, 512, 1024) on retrieval accuracy for Q&A.
3. Implement HyDE (Hypothetical Document Embedding) and measure recall improvement.
4. Evaluate a RAG system using RAGAS faithfulness and relevancy metrics.

---

## Chapter 108: LangChain Framework

### Introduction

Framework for LLM apps: prompts, models, chains, agents, tools, memory. LCEL (| operator) for declarative pipelines.

### Theory

**LCEL:** chain = prompt | model | parser. RunnablePassthrough, RunnableParallel.

**Memory:** Buffer (full history), Window (last k), Summary (compressed), VectorStore (semantic).

**Agents:** ReAct, Tool Calling. AgentExecutor with handle_parsing_errors, max_iterations.

> 💡 **Pro Tip:** LangChain Expression Language (LCEL) is the recommended way to build chains. It provides automatic streaming, async support, and built-in observability via LangSmith. Avoid the legacy `LLMChain` class — it lacks these features and is being deprecated.

### Common Mistakes

- Forgetting memory for conversational agents
- No parsing error handling
- Using LLMChain over LCEL

### Best Practices

1. Use LCEL over LLMChain
2. Add memory for multi-turn
3. handle_parsing_errors=True on AgentExecutor

### Revision Notes

- LCEL: declarative | pipelines
- Memory: Buffer, Window, Summary, Vector
- Agents: ReAct, Tool Calling, AgentExecutor

### Interview Questions

1. What is LCEL and how does it differ from the legacy LLMChain API?
2. Compare the different memory types in LangChain: Buffer, Window, Summary, VectorStore.
3. How does LangChain's AgentExecutor handle parsing errors?
4. What is the role of RunnablePassthrough and RunnableParallel in LCEL?
5. How does LangSmith enable observability in LangChain applications?

### Practical Exercises

1. Build a simple LCEL chain: prompt | model | output parser.
2. Implement a conversational agent with memory using BufferMemory.
3. Create a ReAct agent that can search the web and perform calculations.
4. Use RunnableParallel to execute two chains in parallel and combine results.

---

## Chapter 109: LangGraph

### Introduction

Stateful, cyclic graph framework for multi-agent. Nodes + edges + conditional edges + checkpointing.

### Theory

**StateGraph:** TypedDict state, nodes (functions), edges (transitions), conditional edges (routing).

**Checkpointing:** SqliteSaver (local), PostgresSaver (production). Thread isolation per conversation.

**Human-in-loop:** Pause graph for approval, update_state to resume.

**Multi-agent:** Supervisor (route to specialists), Reflection (generate -> critique), Debate (agents argue).

### Common Mistakes

- Not using reducers for merge behavior
- Forgetting checkpointing
- Infinite cycles (no path to END)

### Best Practices

1. Always use checkpoints in production
2. Thread isolation per conversation
3. Human-in-loop for high-stakes actions

> ✅ **Best Practice:** Always implement checkpointing in LangGraph for production use. It enables fault tolerance, human-in-the-loop approval, and the ability to replay or roll back graph execution. Start with SQLite for development and migrate to PostgreSQL for production.

### Revision Notes

- StateGraph: nodes + edges + conditional edges
- Reducers: define state merge behavior
- Checkpointing: persist state (SQLite/PostgreSQL)
- Multi-agent: Supervisor, Reflection, Debate

### Interview Questions

1. How does LangGraph's StateGraph differ from LangChain's chain-based approach?
2. What are reducers and why are they needed in LangGraph?
3. How does checkpointing enable human-in-the-loop workflows?
4. Compare Supervisor, Reflection, and Debate multi-agent architectures in LangGraph.
5. How would you implement a conditional edge that routes between different nodes?

### Practical Exercises

1. Build a simple state graph with two nodes and a conditional edge.
2. Implement a supervisor agent that routes between specialist agents.
3. Add checkpointing with PostgresSaver and test fault recovery.
4. Create a reflection loop where one agent generates and another critiques.

---

## Chapter 110: LlamaIndex

### Introduction

Data framework for LLM apps. Focus on data ingestion, indexing, retrieval. 100+ connectors, 10+ index types.

### Theory

**Index types:** VectorStoreIndex (semantic), SummaryIndex (flat list), TreeIndex (hierarchical), KeywordTableIndex (keyword), KnowledgeGraphIndex (triples).

**Retrievers:** EnsembleRetriever (hybrid vector + BM25), RecursiveRetriever, RouterRetriever.

**Query Engines:** RetrieverQueryEngine, SubQuestionQueryEngine (decomposition), SQLJoinQueryEngine, PandasQueryEngine.

> 💡 **Pro Tip:** LlamaIndex's SubQuestionQueryEngine is powerful for complex queries — it decomposes a question into sub-questions, answers each independently, then synthesizes the final answer. This dramatically improves accuracy on multi-hop questions compared to simple retrieval.

### Best Practices

1. SentenceSplitter as default
2. Always add re-ranking in production
3. Hybrid retrieval (EnsembleRetriever)

### Revision Notes

- Documents -> Nodes -> Indices -> Retrievers -> Query Engines
- 5 index types: Vector, Summary, Tree, Keyword, KG
- SubQuestionQueryEngine: query decomposition

### Interview Questions

1. Compare the 5 index types in LlamaIndex and when to use each.
2. How does SubQuestionQueryEngine decompose and answer complex queries?
3. What is the role of EnsembleRetriever in hybrid search?
4. How do Nodes differ from Documents in LlamaIndex's data model?
5. When would you use KnowledgeGraphIndex over VectorStoreIndex?

### Practical Exercises

1. Ingest a set of PDFs using LlamaIndex and query them with VectorStoreIndex.
2. Build a hybrid retriever combining vector search and BM25 keyword search.
3. Use SubQuestionQueryEngine to answer a multi-hop question.
4. Compare the accuracy of TreeIndex vs VectorStoreIndex on hierarchical data.


---

## Chapter 111: Hugging Face Ecosystem

### Introduction

Hugging Face is the most comprehensive open-source ML platform. The transformers library provides unified API (AutoModel, pipeline). The datasets library enables efficient data loading. PEFT provides parameter-efficient fine-tuning (LoRA, QLoRA).

### Theory

**Transformers:** AutoModel.from_pretrained(), pipeline() for all modalities (NLP, vision, audio).

**Trainer:** TrainingArguments + Trainer for training loop. Seq2SeqTrainer for encoder-decoder.

**PEFT Methods:** LoRA (low-rank A*B, r=8-128, alpha=16-256), QLoRA (4-bit NF4 + double quantization), AdaLoRA (adaptive rank), IA3 (scaling vectors), Prefix Tuning (prefix tokens), Prompt Tuning (soft prompts).

**TRL:** SFTTrainer (supervised fine-tuning), DPOTrainer (direct preference optimization).

**Inference:** vLLM (PagedAttention, 2-10x throughput), TGI (Hugging Face), TensorRT-LLM, ONNX Runtime.

> ⚠️ **Warning:** When loading large models with `from_pretrained`, always specify `torch_dtype=torch.float16` or `torch.bfloat16` and `device_map="auto"`. Loading in full float32 on a single GPU will cause out-of-memory errors for models larger than ~3B parameters.

### Best Practices

1. Use from_pretrained with torch_dtype and device_map
2. Use datasets in streaming mode for data > RAM
3. Use PEFT (LoRA/QLoRA) for fine-tuning large models

### Revision Notes

- transformers: unified API via from_pretrained and pipeline
- PEFT: LoRA (low-rank), QLoRA (4-bit), AdaLoRA (adaptive)
- TRL: SFTTrainer, DPOTrainer, PPOTrainer
- Inference: vLLM, TGI, TensorRT-LLM

### Interview Questions

1. How does the Hugging Face `pipeline()` API unify different modalities?
2. Compare LoRA, QLoRA, and AdaLoRA in terms of memory, speed, and quality.
3. What is the role of TRL's SFTTrainer and DPOTrainer in fine-tuning?
4. How does `device_map="auto"` work when loading models?
5. Compare vLLM vs TGI for production inference serving.

### Practical Exercises

1. Load a model using from_pretrained with different quantization settings and compare memory usage.
2. Fine-tune a small model (e.g., GPT-2) using LoRA with the SFTTrainer.
3. Use the datasets library in streaming mode to process a dataset larger than RAM.
4. Deploy a model with vLLM and measure throughput with continuous batching.

---

## Chapter 112: Fine-Tuning LLMs

### Introduction

Full fine-tuning (all params) vs parameter-efficient (LoRA, QLoRA). Preference tuning (RLHF, DPO). Data quality and hyperparameters critical.

### Theory

**Full FT:** 8x GPU memory of inference (optimizer + gradients + model + activations). 70B requires ~1120 GB.

**LoRA:** W' = W + (alpha/r) * BA. r=4-128. 0.1-2% trainable params.

**QLoRA:** LoRA on 4-bit NF4. Enables 65B on single 48GB GPU.

**RLHF:** SFT -> Reward Model (Bradley-Terry) -> PPO + KL constraint.

**DPO:** Implicit reward, closed-form, no separate RM.

**Hyperparams:** LR (1e-4 LoRA, 1e-5 full), epochs (1-3), AdamW 8-bit.

> 📖 **Instructor Note:** The key insight of LoRA is that weight updates during fine-tuning have low intrinsic rank. Instead of updating a full weight matrix W (e.g., 4096x4096), LoRA learns two low-rank matrices A and B (e.g., 4096x16 and 16x4096), reducing trainable parameters by 100x while maintaining quality.

### Best Practices

1. Prefer LoRA/QLoRA for most use cases
2. r=16 default, tune up/down
3. Train 1-2 epochs max
4. Use DPO over RLHF for simplicity

### Revision Notes

- Full FT: expensive, high quality
- LoRA: W' = W + (alpha/r) * BA
- QLoRA: 4-bit NF4, double quantization
- RLHF: SFT -> RM -> PPO
- DPO: implicit reward, no RM

### Interview Questions

1. Why does LoRA reduce memory requirements for fine-tuning?
2. How does QLoRA enable fine-tuning 65B models on a single 48GB GPU?
3. Compare RLHF and DPO — what are the tradeoffs?
4. What is the role of the reward model in RLHF?
5. How do you choose the LoRA rank r for different tasks?

### Practical Exercises

1. Fine-tune a 7B model using LoRA on a custom instruction dataset.
2. Compare full fine-tuning vs LoRA fine-tuning in terms of memory, time, and quality.
3. Implement DPO training for preference alignment.
4. Evaluate model quality before and after fine-tuning on domain-specific tasks.

---

## Chapter 113: Local LLM Deployment

### Introduction

Ollama (simplest), llama.cpp (best quantization), vLLM (best throughput), TGI. Open models: LLaMA 3, Mistral, Qwen, DeepSeek, Phi-3.

### Theory

**Ollama:** pull, run, Modelfile (FROM, PARAMETER, TEMPLATE, SYSTEM). OpenAI-compatible API.

**llama.cpp:** GGUF format, K-quants (q4_k_m recommended). Quantization: GPTQ vs AWQ vs bitsandbytes vs GGUF.

**vLLM:** PagedAttention, continuous batching, prefix caching, speculative decoding.

### Tables

| Model | Size | Context | Strengths |
|-------|------|---------|-----------|
| LLaMA 3 | 8B-405B | 8K-128K | Best general |
| Mistral 7B | 7B | 32K | Best 7B |
| Mixtral 8x7B | 46.7B | 32K | MoE, fast |
| Qwen 2.5 | 0.5B-72B | 32K-128K | Multilingual |

> ✅ **Best Practice:** Start with Ollama for local experimentation — it handles model downloading, quantization, and provides an OpenAI-compatible API out of the box. For production, migrate to vLLM which offers PagedAttention for efficient KV cache management and continuous batching for higher throughput.

### Best Practices

1. q4_k_m GGUF as default quantization
2. Ollama for quick local experiments
3. vLLM for production local serving
4. 7B on 8GB, 13B on 16GB, 70B on 48GB (q4)

### Revision Notes

- Ollama: pull, run, Modelfile, OpenAI-compatible
- llama.cpp: GGUF, K-quants, server mode
- vLLM: PagedAttention, continuous batching, prefix caching
- Speculative decoding: draft + target, 2-3x speedup

### Interview Questions

1. How does speculative decoding achieve 2-3x speedup in LLM inference?
2. Compare GPTQ, AWQ, and GGUF quantization methods.
3. What is PagedAttention and how does it improve vLLM performance?
4. How does continuous batching differ from static batching?
5. What are the GPU memory requirements for running 7B, 13B, and 70B models at q4?

### Practical Exercises

1. Install Ollama and run a 7B model locally. Measure tokens/second.
2. Quantize a model to q4_k_m using llama.cpp and compare inference speed vs fp16.
3. Deploy a model with vLLM and measure throughput under concurrent requests.
4. Compare speculative decoding vs standard generation speed for a draft-target model pair.


---

## Chapter 114: Function Calling and Tools

### Introduction

LLMs generate structured JSON for tool execution. Enables real-time data access, computation, API orchestration, structured output.

### Theory

**OpenAI function calling:** tools=[{"type":"function","function":{"name":...,"parameters":{...}}}]. tool_choice: auto/required/none.

**Schema design:** Clear descriptions, enums for constrained values, examples in descriptions, only truly required fields.

**Parallel function calling:** Multiple tools in one response. Execute in parallel.

**Structured output:** JSON mode, Outlines (constrained decoding), Instructor (Pydantic-based).

**MCP (Model Context Protocol):** Open standard. Hosts + Servers (tools/resources/prompts) + Transports (stdio/SSE/WebSocket).

> ⚠️ **Warning:** Function calling schemas with deep nesting or ambiguous descriptions often cause models to hallucinate parameters or omit required fields. Keep schemas flat, provide clear descriptions with example values, and use `required` to specify only truly mandatory parameters.

### Best Practices

1. Clear function descriptions with example triggers
2. Use enums for constrained params
3. Handle errors gracefully (retry, fallback)
4. Keep schemas flat (avoid deep nesting)

### Revision Notes

- Function calling: model outputs JSON for tools
- Schema: name, description, parameters (JSON Schema), required
- tool_choice: auto, required, specific tool
- MCP: open standard for LLM-tool integration
- Structured output: JSON mode, Outlines, Instructor

### Interview Questions

1. How does function calling work in OpenAI's API?
2. What is the Model Context Protocol (MCP) and how does it standardize tool integration?
3. Compare JSON mode, Outlines, and Instructor for structured output generation.
4. How does parallel function calling work and when should you use it?
5. What error handling strategies should you implement for function calling?

### Practical Exercises

1. Define a function schema for a weather API and test the model's ability to call it correctly.
2. Implement parallel function calling with two independent tools.
3. Use Instructor to generate structured Pydantic output from an LLM.
4. Build an MCP server that exposes a tool and connect it to an MCP host.

---

## Chapter 115: AI Agents

### Introduction

LLM + tools + memory + planning. Agent loop: observe -> think -> act -> observe.

### Theory

**ReAct:** Thought -> Action -> Observation -> repeat until done.

**Components:** Tools (search, calculator, code interpreter, APIs), Memory (short-term conversation, long-term vector store, entity knowledge), Planning (task decomposition, sub-goals, replanning).

**Code interpreter:** Sandboxed Python execution with timeout.

**Error handling:** Retry with modified input, fallback tools, self-correction.

> 📖 **Instructor Note:** The ReAct pattern (Reason + Act) is the foundation of most agent architectures. The key insight is interleaving reasoning traces with actions — the model "thinks aloud" about what to do, executes a tool, observes the result, and continues reasoning. This explicit reasoning trace makes agents more robust and debuggable.

### Best Practices

1. Always set max_iterations (5-15)
2. Comprehensive error handling for every tool
3. Input validation and output sanitization
4. Log every agent step for debugging

### Revision Notes

- Agent loop: Observe -> Think -> Act -> Observe
- Components: Tools, Memory, Planning
- ReAct: Thought -> Action -> Observation
- Error handling: retry, fallback, self-correction

### Interview Questions

1. How does the ReAct pattern combine reasoning and action?
2. What are the essential components of an AI agent?
3. How do you handle errors when a tool fails or returns unexpected results?
4. What strategies prevent agents from entering infinite loops?
5. How does planning (task decomposition) improve agent performance on complex tasks?

### Practical Exercises

1. Implement a ReAct agent that can search the web and perform calculations.
2. Build a code interpreter agent with sandboxed Python execution.
3. Add comprehensive error handling and logging to an agent system.
4. Implement task decomposition where an agent breaks a complex query into sub-tasks.

---

## Chapter 116: Multi-Agent Systems

### Introduction

Multiple specialized agents communicating and coordinating. Architectures: supervisor, swarm, debate, pipeline.

### Theory

**Architectures:** Supervisor (one delegates), Swarm (no central control), Debate (argue positions), Voting (majority rule), Pipeline (sequential stages).

**Communication:** Shared state, message passing, event bus.

**Frameworks:** AutoGen (GroupChat, AssistantAgent), CrewAI (Agent, Task, Crew, Process), MetaGPT (PM, Architect, Engineer, QA).

> 💡 **Pro Tip:** For multi-agent systems, start with a simple Supervisor architecture before moving to more complex patterns like Swarm or Debate. Supervisor agents are easier to debug, have well-defined handoffs, and are sufficient for most real-world applications. Add complexity only when your use case demands it.

### Best Practices

1. Define clear roles for each agent
2. Use shared state for coordination
3. Limit context per agent
4. Implement verification: one produces, another validates

### Revision Notes

- Architectures: Supervisor, Swarm, Debate, Voting, Pipeline
- Communication: shared state, message passing, event bus
- AutoGen: GroupChat, AssistantAgent, UserProxyAgent
- CrewAI: Agent, Task, Crew, Process

### Interview Questions

1. Compare Supervisor, Swarm, and Debate multi-agent architectures.
2. How do agents communicate and share state in a multi-agent system?
3. What is the role of a UserProxyAgent in AutoGen?
4. How does CrewAI define tasks and assign them to agents?
5. When would you choose a Pipeline architecture over a Supervisor architecture?

### Practical Exercises

1. Build a supervisor agent that delegates tasks to specialist agents.
2. Implement a debate system where two agents argue different positions and a judge decides.
3. Use AutoGen to create a GroupChat with three specialist agents.
4. Build a multi-agent pipeline where each agent processes the output of the previous one.

---

## Chapter 117: AI Agent Frameworks

### Introduction

LangGraph (state graphs), AutoGen (conversational), CrewAI (role-based), Semantic Kernel (enterprise), Assistants API (managed), DSPy (programmatic).

### Theory

**DSPy:** Signatures, modules, optimizers. Programmatic prompting — compile optimizes prompts.

**Assistants API:** Thread-based, code interpreter, file search, function calling.

**Semantic Kernel:** Planners, connectors, plugins, memory for .NET applications.

### Framework Comparison

| Framework | Focus | Best For |
|-----------|-------|----------|
| LangGraph | Stateful graphs | Complex workflows |
| AutoGen | Multi-agent chat | Conversations |
| CrewAI | Role-based teams | Simple delegation |
| DSPy | Prompt optimization | Programmatic prompts |

> ✅ **Best Practice:** Choose your agent framework based on your workflow complexity. For linear chains, use LangChain. For cyclic stateful graphs, use LangGraph. For conversational multi-agent, use AutoGen. For programmatic prompt optimization, use DSPy. Don't over-architect — start simple and upgrade as needed.

### Best Practices

1. Start with simplest framework for your problem
2. LangGraph for complex stateful workflows
3. AutoGen for multi-agent conversations
4. DSPy when optimizing prompts programmatically

### Revision Notes

- LangGraph: state graphs, cyclic, checkpointing
- AutoGen: GroupChat, code exec, AssistantAgent
- CrewAI: Agent, Task, Crew, Process
- DSPy: signatures, modules, optimizers

### Interview Questions

1. Compare LangGraph, AutoGen, and CrewAI framework philosophies.
2. How does DSPy's programmatic prompting differ from manual prompt engineering?
3. What is the role of optimizers in DSPy?
4. How does the Assistants API differ from open-source agent frameworks?
5. When would you use Semantic Kernel vs LangChain for enterprise applications?

### Practical Exercises

1. Build the same agent workflow in LangGraph and AutoGen and compare complexity.
2. Use DSPy to optimize a prompt for a classification task.
3. Create a CrewAI pipeline with Agent, Task, Crew, and Process definitions.
4. Benchmark two different frameworks on the same multi-agent task.


---

## Chapter 118: AI Safety and Alignment

### Introduction

Ensuring AI systems behave according to human intent. Pillars: Helpfulness, Harmlessness, Honesty (HHH).

### Theory

**RLHF:** Human preferences -> Reward Model (Bradley-Terry) -> PPO + KL constraint.

**Constitutional AI (Anthropic):** Self-critique and revision using a constitution.

**Red teaming:** Jailbreaks (DAN, role-playing, hypothetical, encoding, multi-language, CoT). Prompt injection (direct user, indirect content).

**Defenses:** Input/output filtering, system prompt hardening, instruction hierarchy, perplexity detection.

**Guardrails:** NeMo Guardrails (topic/safety/dialog/execution rails), Guardrails AI (validators), Llama Guard, Azure AI Content Safety, Presidio (PII).

**Hallucination:** Causes (data compression, sampling, knowledge cutoff). Detection (self-consistency, RAG, calibration). Mitigation (RAG, constrained decoding, low temperature).

**Bias:** Representational harm, demographic bias, allocation harm. Evaluation: WinoBias, BOLD, CrowS-Pairs, StereoSet.

> ⚠️ **Warning:** Prompt injection is one of the most critical security risks in LLM applications. In direct injection, a user crafts input that overrides system instructions. In indirect injection, untrusted content (e.g., a retrieved web page) contains injection payloads. Always implement multi-layer defenses: system prompt hardening, input/output filtering, and guardrails.

### Best Practices

1. Multi-layer defense: prompt + guardrails + filtering
2. Red team before deployment
3. Monitor for new jailbreak patterns
4. Evaluate bias across demographic groups
5. Instruction hierarchy: system > user > tool

### Revision Notes

- Alignment: Helpful, Harmless, Honest (HHH)
- RLHF: RM -> PPO + KL; Constitutional AI: self-critique
- Red teaming: adversarial testing, jailbreaks
- Prompt injection: direct vs indirect
- Guardrails: NeMo, Guardrails AI, Llama Guard
- Hallucination: RAG, self-consistency, calibration
- Bias: WinoBias, BOLD, CrowS-Pairs

### Interview Questions

1. What is the difference between direct and indirect prompt injection?
2. How does Constitutional AI enable self-critique and revision?
3. What strategies can mitigate hallucination in LLM outputs?
4. How do you evaluate bias in LLM outputs across demographic groups?
5. What is the instruction hierarchy and why is it important for security?

### Practical Exercises

1. Implement input/output filtering to detect and block prompt injection attempts.
2. Set up NeMo Guardrails for topic control and safety filtering.
3. Red team an LLM with common jailbreak techniques and document success rates.
4. Evaluate a model's bias using WinoBias or BOLD datasets.

---

## Chapter 119: Evaluation of LLMs and RAG Systems

### Introduction

Intrinsic (perplexity), extrinsic (BLEU/ROUGE), semantic (BERTScore), RAG-specific (RAGAS), LLM-as-judge, human evaluation.

### Theory

**Metrics:** Perplexity (arch comparison), BLEU (precision n-gram), ROUGE (recall n-gram), BERTScore (embedding-based), RAGAS (faithfulness, relevancy, precision, recall).

**LLM-as-judge:** MT-Bench, AlpacaEval, Arena Hard, Auto-J. Biases: position, verbosity, self-enhancement.

**Human evaluation:** Chatbot Arena (LMSYS Elo) — 100K+ pairwise comparisons.

**Bias & toxicity:** Perspective API, Detoxify, Llama Guard, WinoBias, BOLD, CrowS-Pairs.

> 💡 **Pro Tip:** LLM-as-judge is efficient but has well-known biases: it favors longer responses (length bias), answers in certain positions (position bias), and responses that mirror its own style (self-enhancement bias). Always control for these by randomizing presentation order and using reference answers.

### Best Practices

1. Use multiple metrics: intrinsic + extrinsic + semantic
2. Always evaluate RAG faithfulness
3. Control for position/length bias in LLM-as-judge
4. 500+ samples for statistical significance
5. Evaluate on YOUR specific task

### Revision Notes

- Intrinsic: Perplexity = exp(-1/N sum log P(w_i))
- Extrinsic: BLEU, ROUGE, METEOR
- Semantic: BERTScore, MoverScore
- RAGAS: Faithfulness, Relevancy, Precision, Recall
- LLM-as-judge: MT-Bench, AlpacaEval, biases
- Human eval: Chatbot Arena Elo

### Interview Questions

1. What is the difference between intrinsic and extrinsic evaluation metrics?
2. How does RAGAS measure faithfulness and answer relevancy?
3. What biases exist in LLM-as-judge evaluation and how do you mitigate them?
4. Why is perplexity insufficient for evaluating generative models?
5. How does Chatbot Arena's Elo rating system work for model comparison?

### Practical Exercises

1. Compute perplexity, BLEU, ROUGE, and BERTScore for a set of generated texts.
2. Implement RAGAS evaluation for a RAG pipeline and identify faithfulness failures.
3. Use an LLM-as-judge to evaluate responses and control for position bias.
4. Build an automated evaluation suite that runs multiple metrics on model outputs.

---

## Chapter 120: Production LLM Systems

### Introduction

Architecture: Frontend -> API Gateway -> Load Balancer -> App Server -> LLM Service -> RAG Pipeline.

### Theory

**vLLM:** PagedAttention, continuous batching, prefix caching. 2-10x throughput.

**Scaling:** Vertical (bigger GPU), Horizontal (more instances), Tensor Parallel (intra-layer), Pipeline Parallel (inter-layer), FSDP/ZeRO (sharded).

**Caching:** Exact (Redis), Semantic (embedding similarity), KV (per-session).

**Cost optimization:** Model routing (small->large), prompt compression, caching, batching, spot instances.

**Monitoring:** TTFT (time-to-first-token), TPOT (time-per-output-token), latency p99, error rate, cost/query, cache hit rate.

**Deployment:** Docker + Kubernetes + Helm + HPA/KEDA auto-scaling + GPU node pools.

> ✅ **Best Practice:** Model routing is one of the most cost-effective optimizations for production LLM systems. Route simple queries (translation, summarization) to small, cheap models and complex queries (reasoning, code generation) to large models. This can reduce costs by 60-80% while maintaining quality for most users.

### Best Practices

1. Multi-layer caching (exact + semantic + KV)
2. Model routing: small for simple, large for complex
3. Monitor: TTFT, TPOT, error rate, cache hit, cost
4. Auto-scale based on queue depth or GPU utilization
5. A/B test model changes before full rollout

### Revision Notes

- Architecture: FE -> API Gateway -> App -> LLM -> RAG
- vLLM: PagedAttention, continuous batching, 2-10x throughput
- Scaling: Vertical, Horizontal, TP/PP/FSDP
- Caching: Exact (Redis), Semantic (embedding), KV (session)
- Cost: Model routing, compression, caching, batching
- Monitoring: TTFT, TPOT, latency, error rate, cost, cache hit
- Deployment: Docker, K8s, Helm, HPA/KEDA

### Interview Questions

1. What are the key latency metrics (TTFT, TPOT) and how do you optimize each?
2. How does model routing reduce costs in production LLM systems?
3. Compare tensor parallelism, pipeline parallelism, and FSDP for multi-GPU deployment.
4. How would you design a caching strategy for a production LLM application?
5. What monitoring metrics are essential for production LLM systems?

### Practical Exercises

1. Design a production architecture for an LLM application with caching, routing, and auto-scaling.
2. Calculate the cost per query for different model sizes and routing strategies.
3. Implement a simple model router that sends queries to different models based on complexity.
4. Set up monitoring for TTFT, TPOT, and cache hit rate using Prometheus metrics.

---

## Appendix A: Cross-References

| Concept | See Also |
|---------|----------|
| Transformer Architecture | Part 13 |
| Attention Mechanisms | Part 13 Ch 92 |
| Distributed Training | Part 12 Ch 87-88 |
| Deep Learning | Part 11 |
| Python | Part 1-3 |
| MLOps / CI/CD | Part 12 Ch 89 |
| Docker / Kubernetes | Part 12 |
| Probability & Statistics | Part 8 |

## Appendix B: Recommended Reading

**Papers:** Attention Is All You Need, Scaling Laws (Kaplan), Chinchilla Scaling Laws (Hoffmann), Chain-of-Thought (Wei), Tree of Thoughts (Yao), ReAct (Yao), RAG (Lewis), Self-RAG (Asai), LoRA (Hu), QLoRA (Dettmers), DPO (Rafailov), Constitutional AI (Bai), LLaMA (Touvron), Mistral 7B (Jiang), Mixtral of Experts (Jiang).

**Books:** Speech and Language Processing (Jurafsky & Martin), Deep Learning (Goodfellow et al.), NLP with Transformers (Tunstall et al.).

**Libraries:** Hugging Face Transformers/Datasets/PEFT/TRL, LangChain/LangGraph/LangSmith, LlamaIndex, vLLM/TGI/llama.cpp, FAISS/Qdrant/Milvus/Chroma/Pinecone, DSPy/AutoGen/CrewAI, RAGAS/TruLens.

**Courses:** Stanford CS224N, Hugging Face NLP Course, DeepLearning.AI Short Courses, Kilo Full-Stack AI Roadmap.

---

*End of Part 14: Generative AI*

---

## Expanded Content: Chapters 101-103

### Chapter 101: Generative AI — Deep Dive

#### Code Example: Implementing a Simple VAE

`python
import torch
import torch.nn as nn
import torch.nn.functional as F

class VAE(nn.Module):
    def __init__(self, input_dim=784, latent_dim=32, hidden_dim=256):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
        )
        self.mu = nn.Linear(hidden_dim, latent_dim)
        self.logvar = nn.Linear(hidden_dim, latent_dim)
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, input_dim),
            nn.Sigmoid(),
        )

    def encode(self, x):
        h = self.encoder(x)
        return self.mu(h), self.logvar(h)

    def reparameterize(self, mu, logvar):
        std = torch.exp(0.5 * logvar)
        eps = torch.randn_like(std)
        return mu + eps * std

    def decode(self, z):
        return self.decoder(z)

    def forward(self, x):
        mu, logvar = self.encode(x)
        z = self.reparameterize(mu, logvar)
        recon = self.decode(z)
        return recon, mu, logvar

def vae_loss(recon, x, mu, logvar):
    recon_loss = F.binary_cross_entropy(recon, x, reduction='sum')
    kl_loss = -0.5 * torch.sum(1 + logvar - mu.pow(2) - logvar.exp())
    return recon_loss + kl_loss
`

#### Scaling Laws — Detailed Formulation

The scaling law from Kaplan et al. (2022) states:

L(N, D) = \left(\frac{N_c}{N}\right)^{\alpha_N} + \left(\frac{D_c}{D}\right)^{\alpha_D} + L_\infty

Where N_c and D_c are constants, alpha_N ≈ 0.076, alpha_D ≈ 0.095, and L_inf ≈ 1.69 (for the test set used).

| Model Size | Compute Budget | Optimal Data Ratio | Achievable Loss |
|------------|---------------|-------------------|-----------------|
| 1B params | 1e20 FLOP | 20B tokens | 3.2 |
| 10B params | 1e21 FLOP | 200B tokens | 2.8 |
| 100B params | 1e22 FLOP | 2T tokens | 2.4 |
| 1T params | 1e23 FLOP | 20T tokens | 2.0 |

### Chapter 102: LLM Architecture — Expanded

#### Attention Mechanism Comparison

`python
import torch
import torch.nn as nn
import torch.nn.functional as F

class MultiHeadAttention(nn.Module):
    def __init__(self, dim, num_heads, num_kv_heads=None):
        super().__init__()
        self.num_heads = num_heads
        self.num_kv_heads = num_kv_heads or num_heads  # GQA: fewer KV heads
        self.head_dim = dim // num_heads
        self.num_queries_per_kv = num_heads // self.num_kv_heads

        self.q_proj = nn.Linear(dim, num_heads * self.head_dim, bias=False)
        self.k_proj = nn.Linear(dim, self.num_kv_heads * self.head_dim, bias=False)
        self.v_proj = nn.Linear(dim, self.num_kv_heads * self.head_dim, bias=False)
        self.o_proj = nn.Linear(num_heads * self.head_dim, dim, bias=False)

    def forward(self, x, mask=None):
        B, T, D = x.shape
        q = self.q_proj(x).view(B, T, self.num_heads, self.head_dim).transpose(1, 2)
        k = self.k_proj(x).view(B, T, self.num_kv_heads, self.head_dim).transpose(1, 2)
        v = self.v_proj(x).view(B, T, self.num_kv_heads, self.head_dim).transpose(1, 2)

        # Expand KV heads for GQA
        if self.num_kv_heads != self.num_heads:
            k = k.repeat_interleave(self.num_queries_per_kv, dim=1)
            v = v.repeat_interleave(self.num_queries_per_kv, dim=1)

        scores = torch.matmul(q, k.transpose(-2, -1)) / (self.head_dim ** 0.5)
        if mask is not None:
            scores = scores + mask
        attn = F.softmax(scores, dim=-1)
        out = torch.matmul(attn, v)
        out = out.transpose(1, 2).contiguous().view(B, T, -1)
        return self.o_proj(out)

# GQA: 32 query heads, 8 KV heads -> 4x KV cache reduction
# MHA: H query + H KV heads -> largest KV cache
# MQA: H query + 1 KV head -> smallest KV cache, slight quality loss
`

#### KV Cache Memory Calculation

`python
def compute_kv_cache_memory(model_config, sequence_length, batch_size=1, dtype_bytes=2):
    \"\"\"Calculate KV cache memory requirements.\"\"\"
    n_layers = model_config["n_layer"]
    n_kv_heads = model_config.get("n_kv_heads", model_config["n_head"])
    head_dim = model_config["n_embd"] // model_config["n_head"]

    # KV cache stores keys and values for each layer
    # Size: 2 (K+V) * batch * seq_len * n_kv_heads * head_dim * dtype_bytes
    per_layer = 2 * batch_size * sequence_length * n_kv_heads * head_dim * dtype_bytes
    total = per_layer * n_layers

    return total / (1024**3)  # GB

# LLaMA 2 70B: 80 layers, 64 query heads, 8 KV heads, 8192 dim
configs = {
    "LLaMA 2 70B (MHA)": {"n_layer": 80, "n_head": 64, "n_kv_heads": 64, "n_embd": 8192},
    "LLaMA 2 70B (GQA)": {"n_layer": 80, "n_head": 64, "n_kv_heads": 8, "n_embd": 8192},
}
for name, cfg in configs.items():
    mem = compute_kv_cache_memory(cfg, 4096)
    print(f"{name}: {mem:.2f} GB at 4K context")
    # MHA: 163.84 GB, GQA: 20.48 GB (8x reduction)
`

### Chapter 103: Tokenization — Expanded

#### Training a Custom BPE Tokenizer

`python
from tokenizers import Tokenizer, models, trainers, pre_tokenizers, decoders

# Step 1: Initialize BPE model
tokenizer = Tokenizer(models.BPE())

# Step 2: Set pre-tokenizer (how to split initial text)
tokenizer.pre_tokenizer = pre_tokenizers.ByteLevel(add_prefix_space=True)

# Step 3: Configure trainer
trainer = trainers.BpeTrainer(
    vocab_size=32000,
    special_tokens=["<s>", "<pad>", "</s>", "<unk>", "<mask>"],
    min_frequency=2,
    show_progress=True,
    initial_alphabet=pre_tokenizers.ByteLevel.alphabet(),
)

# Step 4: Train on files
files = ["data/train_en.txt", "data/train_code.py", "data/train_ja.txt"]
tokenizer.train(files, trainer)

# Step 5: Post-process and decode
tokenizer.post_processor = processors.ByteLevel(trim_offsets=True)
tokenizer.decoder = decoders.ByteLevel()

# Step 6: Save and load
tokenizer.save("my_bpe_tokenizer.json")
loaded = Tokenizer.from_file("my_bpe_tokenizer.json")

# Test
output = loaded.encode("Hello, world! Transformer models are amazing.")
print(f"Tokens: {output.tokens}")
print(f"IDs: {output.ids}")
print(f"Length: {len(output.ids)} tokens vs {len('Hello, world!')} chars")
`

#### Token-to-Character Ratios by Language

`python
# Comparing GPT-4 tokenizer efficiency across languages
import tiktoken

enc = tiktoken.get_encoding("cl100k_base")
texts = {
    "English": "The quick brown fox jumps over the lazy dog.",
    "Spanish": "El zorro marron rapido salta sobre el perro perezoso.",
    "Mandarin": "敏捷的棕色狐狸跳过了懒狗。",
    "Japanese": "素早い茶色の狐が怠け者の犬を飛び越える。",
    "Arabic": "الثعلب البني السريع يقفز فوق الكلب الكسول.",
    "Python Code": "def fibonacci(n): return n if n <= 1 else fibonacci(n-1) + fibonacci(n-2)",
}

for lang, text in texts.items():
    tokens = enc.encode(text)
    ratio = len(text) / len(tokens)
    print(f"{lang:15s}: {len(text):4d} chars -> {len(tokens):4d} tokens = {ratio:.2f} chars/token")
`

| Language | GPT-4 (cl100k) | LLaMA 3 (128K) | LLaMA 2 (32K) |
|----------|---------------|----------------|---------------|
| English | 3.8 chars/tok | 4.1 chars/tok | 4.0 chars/tok |
| Spanish | 3.5 | 3.8 | 3.7 |
| Mandarin | 1.5 | 1.8 | 1.3 |
| Japanese | 1.3 | 1.6 | 1.1 |
| Python | 2.8 | 3.2 | 2.5 |


---

## Expanded Content: Chapters 104-107

### Chapter 104: Prompt Engineering — Advanced Techniques

#### Complete ReAct Agent Implementation

`python
import json
import re
from openai import OpenAI

class ReActAgent:
    def __init__(self, model="gpt-4o", max_steps=10):
        self.client = OpenAI()
        self.model = model
        self.max_steps = max_steps
        self.tools = {}

    def register_tool(self, name, func, description):
        self.tools[name] = {"func": func, "description": description}

    def run(self, question):
        messages = [{"role": "system", "content": """You are a ReAct agent. Use Thought, Action, Observation format.
Available tools: """ + str({k: v["description"] for k, v in self.tools.items()}) + """
When done, respond with: Answer: [final answer]"""}]
        messages.append({"role": "user", "content": question})

        for step in range(self.max_steps):
            response = self.client.chat.completions.create(
                model=self.model, messages=messages, temperature=0
            )
            output = response.choices[0].message.content
            print(f"Step {step+1}: {output[:100]}...")
            messages.append({"role": "assistant", "content": output})

            if output.startswith("Answer:"):
                return output.replace("Answer:", "").strip()

            # Parse action
            action_match = re.search(r"Action: (\w+)\[(.*?)\]", output)
            if action_match:
                tool_name = action_match.group(1)
                tool_input = action_match.group(2)
                if tool_name in self.tools:
                    try:
                        result = self.tools[tool_name]["func"](tool_input)
                        messages.append({"role": "system", "content": f"Observation: {result}"})
                    except Exception as e:
                        messages.append({"role": "system", "content": f"Observation: Error: {e}"})
                else:
                    messages.append({"role": "system", "content": f"Observation: Tool {tool_name} not found."})

        return "Max steps reached without answer."

# Usage
agent = ReActAgent()
agent.register_tool("search", lambda q: f"Results for {q}", "Web search for information")
agent.register_tool("calculator", lambda e: str(eval(e)), "Evaluate math expressions")
result = agent.run("What is 15 * 24 plus the population of France?")
`

#### Structured Prompt Templates

`python
from dataclasses import dataclass
from typing import Optional, List

@dataclass
class SystemPrompt:
    role: str
    instruction: str
    context: Optional[str] = None
    output_format: Optional[str] = None
    constraints: Optional[List[str]] = None

    def build(self) -> str:
        parts = [f"Role: {self.role}", f"Instruction: {self.instruction}"]
        if self.context:
            parts.append(f"Context: {self.context}")
        if self.output_format:
            parts.append(f"Output Format: {self.output_format}")
        if self.constraints:
            parts.append("Constraints:")
            for c in self.constraints:
                parts.append(f"- {c}")
        return "\n\n".join(parts)

prompt = SystemPrompt(
    role="Senior data scientist specializing in NLP",
    instruction="Analyze the sentiment and key topics in the following customer review.",
    context="This is a review for a SaaS product. The customer has been using it for 6 months.",
    output_format="JSON: {sentiment: str, score: float, topics: list[str], summary: str}",
    constraints=["Respond only with valid JSON", "Score between 0 and 1"]
)
`

### Chapter 105: Embeddings — Practical Work

`python
from sentence_transformers import SentenceTransformer, util
import numpy as np

model = SentenceTransformer('all-MiniLM-L6-v2')

# Semantic search
corpus = [
    "The cat sat on the mat",
    "Dogs are playing in the yard",
    "The stock market rallied today",
    "Investors are optimistic about AI companies",
    "Neural networks are inspired by the brain",
    "Python is used for machine learning",
]

corpus_embeddings = model.encode(corpus, convert_to_tensor=True)

def semantic_search(query, top_k=3):
    query_emb = model.encode(query, convert_to_tensor=True)
    scores = util.cos_sim(query_emb, corpus_embeddings)[0]
    results = torch.topk(scores, k=top_k)
    return [(corpus[idx], scores[idx].item()) for idx in results.indices]

# Verify the classic analogy
words = ["king", "queen", "man", "woman", "paris", "france", "berlin", "germany"]
word_embeddings = {w: model.encode(w) for w in words}

# king - man + woman = queen
result = word_embeddings["king"] - word_embeddings["man"] + word_embeddings["woman"]
scores = util.cos_sim(result, word_embeddings["queen"])
print(f"Queen similarity: {scores.item():.3f}")  # Should be ~0.85

# paris - france + germany = berlin
result = word_embeddings["paris"] - word_embeddings["france"] + word_embeddings["germany"]
scores = util.cos_sim(result, word_embeddings["berlin"])
print(f"Berlin similarity: {scores.item():.3f}")  # Should be ~0.75
`

### Chapter 106: Vector Databases — FAISS in Depth

`python
import faiss
import numpy as np

# Generate test data
d = 768  # embedding dimension
n = 100000  # number of vectors
np.random.seed(42)
xb = np.random.random((n, d)).astype(np.float32)
xq = np.random.random((1, d)).astype(np.float32)

# Normalize for cosine similarity
faiss.normalize_L2(xb)
faiss.normalize_L2(xq)

# Build different index types
indexes = {
    "Flat (exact)": faiss.IndexFlatIP(d),
    "IVF4096,Flat": faiss.IndexIVFFlat(faiss.IndexFlatIP(d), d, 4096, faiss.METRIC_INNER_PRODUCT),
    "IVF4096,PQ32": faiss.IndexIVFPQ(faiss.IndexFlatIP(d), d, 4096, 32, 8),
    "HNSW32,Flat": faiss.IndexHNSWFlat(d, 32),
}

for name, index in indexes.items():
    if "IVF" in name or name == "HNSW32,Flat" and "Flat" not in name.split(",")[-1]:
        index.train(xb)
    index.add(xb)

# Benchmark
for name, index in indexes.items():
    if hasattr(index, 'nprobe'):
        index.nprobe = 10

    import time
    start = time.time()
    D, I = index.search(xq, 10)
    elapsed = time.time() - start

    # Calculate recall (using Flat results as ground truth)
    if name == "Flat (exact)":
        truth = set(I[0])
    else:
        recall = len(set(I[0]) & truth) / 10
        print(f"{name:25s}: {elapsed*1000:.1f}ms, recall@{len(truth)} = {recall:.2%}")
`

### Chapter 107: Complete RAG Pipeline with Evaluation

`python
from langchain_chroma import Chroma
from langchain_community.embeddings import SentenceTransformerEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.chains import RetrievalQA
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate

import hashlib

class RAGPipeline:
    def __init__(self, documents, chunk_size=512, chunk_overlap=50, k=5):
        self.embeddings = SentenceTransformerEmbeddings(model_name="all-MiniLM-L6-v2")
        self.splitter = RecursiveCharacterTextSplitter(chunk_size=chunk_size, chunk_overlap=chunk_overlap)
        self.chunks = self.splitter.split_documents(documents)
        self.vector_store = Chroma.from_documents(self.chunks, self.embeddings)
        self.retriever = self.vector_store.as_retriever(search_kwargs={"k": k})
        self.llm = ChatOpenAI(model="gpt-4o", temperature=0)

        self.prompt = ChatPromptTemplate.from_messages([
            ("system", "Answer using ONLY the context. Cite passages. Context: {context}"),
            ("human", "{question}"),
        ])

        self.qa = RetrievalQA.from_chain_type(
            llm=self.llm, chain_type="stuff",
            retriever=self.retriever,
            chain_type_kwargs={"prompt": self.prompt},
            return_source_documents=True,
        )

    def query(self, question):
        result = self.qa.invoke({"query": question})
        sources = [doc.metadata.get("source", "unknown") for doc in result["source_documents"]]
        return {
            "answer": result["result"],
            "sources": sources,
            "num_sources": len(sources),
        }

    def evaluate_ragas(self, questions, ground_truths):
        from ragas import evaluate
        from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall

        dataset = {"question": [], "answer": [], "contexts": [], "ground_truth": []}
        for q, gt in zip(questions, ground_truths):
            result = self.query(q)
            dataset["question"].append(q)
            dataset["answer"].append(result["answer"])
            dataset["contexts"].append([c.page_content for c in result["source_documents"]])
            dataset["ground_truth"].append(gt)

        scores = evaluate(dataset, metrics=[faithfulness, answer_relevancy, context_precision, context_recall])
        return scores
`


---

## Expanded Content: Chapters 108-113

### Chapter 108: LangChain — LCEL Deep Dive

`python
from langchain_core.runnables import RunnablePassthrough, RunnableParallel, RunnableLambda
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate

model = ChatOpenAI(model="gpt-4o", temperature=0)

# Complex LCEL pipeline with branching
def analyze_sentiment(text):
    prompt = ChatPromptTemplate.from_template("Classify sentiment of: {text}")
    chain = prompt | model | StrOutputParser()
    return chain.invoke({"text": text})

def extract_keywords(text):
    prompt = ChatPromptTemplate.from_template("Extract keywords from: {text}")
    chain = prompt | model | StrOutputParser()
    return chain.invoke({"text": text})

def summarize(text):
    prompt = ChatPromptTemplate.from_template("Summarize: {text}")
    chain = prompt | model | StrOutputParser()
    return chain.invoke({"text": text})

# Parallel execution with different analyses
pipeline = RunnableParallel({
    "sentiment": RunnableLambda(analyze_sentiment),
    "keywords": RunnableLambda(extract_keywords),
    "summary": RunnableLambda(summarize),
})

# Combine results
def combine(results):
    return f"""Analysis Report:
- Sentiment: {results['sentiment']}
- Keywords: {results['keywords']}
- Summary: {results['summary']}"""

full_chain = pipeline | RunnableLambda(combine)

result = full_chain.invoke("The new product launch exceeded all expectations. Customers love the AI features.")
print(result)
`

### Chapter 109: LangGraph — Multi-Agent Workflow

`python
from langgraph.graph import StateGraph, END
from typing import TypedDict, List, Annotated
import operator

class CodingState(TypedDict):
    messages: Annotated[List[dict], operator.add]
    task: str
    code: str
    review: str
    iterations: int
    approved: bool

def developer(state: CodingState) -> CodingState:
    prompt = f"Write Python code for: {state['task']}"
    response = llm.invoke(prompt)
    return {"messages": [response], "code": response.content, "iterations": state.get("iterations", 0) + 1}

def reviewer(state: CodingState) -> CodingState:
    prompt = f"Review this code and suggest improvements:\n{state['code']}"
    response = llm.invoke(prompt)
    return {"messages": [response], "review": response.content}

def should_continue(state: CodingState) -> str:
    if state.get("iterations", 0) >= 3:
        return "finalize"
    if "APPROVED" in state.get("review", ""):
        return "finalize"
    return "revise"

def finalizer(state: CodingState) -> CodingState:
    return {"messages": [{"role": "system", "content": "Code finalized."}], "approved": True}

graph = StateGraph(CodingState)
graph.add_node("developer", developer)
graph.add_node("reviewer", reviewer)
graph.add_node("finalizer", finalizer)
graph.set_entry_point("developer")
graph.add_edge("developer", "reviewer")
graph.add_conditional_edges("reviewer", should_continue, {
    "revise": "developer",
    "finalize": "finalizer",
})
graph.add_edge("finalizer", END)

app = graph.compile()
result = app.invoke({"task": "Write a function to merge two sorted lists", "messages": []})
`

### Chapter 110: LlamaIndex — Advanced Indexing

`python
from llama_index.core import (
    VectorStoreIndex, SummaryIndex, TreeIndex, KeywordTableIndex,
    KnowledgeGraphIndex, SimpleDirectoryReader, Settings
)
from llama_index.core.node_parser import (
    SentenceSplitter, SentenceWindowNodeParser, HierarchicalNodeParser,
)
from llama_index.core.postprocessor import SentenceTransformerRerank

# Hierarchical parsing for long documents
parser = HierarchicalNodeParser.from_defaults(chunk_sizes=[2048, 512, 128])
nodes = parser.get_nodes_from_documents(documents)

# Multiple index types for different access patterns
vector_index = VectorStoreIndex(nodes)
summary_index = SummaryIndex(nodes)
tree_index = TreeIndex(nodes)

# Sub-question query engine for complex queries
from llama_index.core.tools import QueryEngineTool, ToolMetadata
from llama_index.core.query_engine import SubQuestionQueryEngine

query_engine = SubQuestionQueryEngine.from_defaults(
    query_engine_tools=[
        QueryEngineTool.from_defaults(
            query_engine=vector_index.as_query_engine(),
            metadata=ToolMetadata(name="vector_search", description="Semantic search"),
        ),
        QueryEngineTool.from_defaults(
            query_engine=summary_index.as_query_engine(),
            metadata=ToolMetadata(name="summarize", description="Document summarization"),
        ),
    ],
    llm=Settings.llm,
)

response = query_engine.query("Compare the key findings from both documents and summarize the methodology.")
`

### Chapter 111: Hugging Face — Training Pipeline

`python
from transformers import (
    AutoModelForSequenceClassification, AutoTokenizer,
    TrainingArguments, Trainer, EarlyStoppingCallback,
)
from datasets import load_dataset
import evaluate
import numpy as np

# Load dataset
dataset = load_dataset("imdb")
tokenizer = AutoTokenizer.from_pretrained("distilbert-base-uncased")

def tokenize_function(examples):
    return tokenizer(examples["text"], padding="max_length", truncation=True, max_length=512)

tokenized_datasets = dataset.map(tokenize_function, batched=True)

# Load model
model = AutoModelForSequenceClassification.from_pretrained("distilbert-base-uncased", num_labels=2)

# Metric
accuracy = evaluate.load("accuracy")
def compute_metrics(eval_pred):
    logits, labels = eval_pred
    predictions = np.argmax(logits, axis=-1)
    return accuracy.compute(predictions=predictions, references=labels)

# Training arguments
training_args = TrainingArguments(
    output_dir="./results",
    evaluation_strategy="epoch",
    save_strategy="epoch",
    learning_rate=2e-5,
    per_device_train_batch_size=16,
    per_device_eval_batch_size=16,
    num_train_epochs=3,
    weight_decay=0.01,
    load_best_model_at_end=True,
    metric_for_best_model="accuracy",
    logging_dir="./logs",
    report_to="wandb",
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_datasets["train"].select(range(2000)),  # Subset for speed
    eval_dataset=tokenized_datasets["test"].select(range(500)),
    compute_metrics=compute_metrics,
    callbacks=[EarlyStoppingCallback(early_stopping_patience=2)],
)

trainer.train()
`

### Chapter 112: Fine-Tuning — Full Pipeline

`python
# Complete LoRA fine-tuning with evaluation
from peft import LoraConfig, get_peft_model, PeftModel
from trl import SFTTrainer, DataCollatorForCompletionOnlyLM

# Prepare base model
base_model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3.2-8B",
    torch_dtype=torch.bfloat16,
    device_map="auto",
    load_in_4bit=True,
)

# Apply LoRA
lora_config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj", "k_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM",
)
model = get_peft_model(base_model, lora_config)
print(model.print_trainable_parameters())

# Training
trainer = SFTTrainer(
    model=model,
    train_dataset=dataset,
    dataset_text_field="text",
    max_seq_length=2048,
    tokenizer=tokenizer,
    args=TrainingArguments(
        output_dir="./lora-finetuned",
        per_device_train_batch_size=4,
        gradient_accumulation_steps=4,
        warmup_ratio=0.03,
        num_train_epochs=2,
        learning_rate=2e-4,
        fp16=True,
        logging_steps=10,
        save_steps=200,
        report_to="wandb",
    ),
    formatting_func=lambda x: [f"### Instruction: {x['instruction']}\\n### Input: {x.get('input', '')}\\n### Response: {x['output']}"],
)

trainer.train()

# Merge and save
merged_model = model.merge_and_unload()
merged_model.save_pretrained("./final-model")
tokenizer.save_pretrained("./final-model")
`

### Chapter 113: Local Deployment — End-to-End

`ash
# Complete local deployment workflow

# 1. Download model
ollama pull llama3.2:8b-instruct-q4_K_M

# 2. Custom Modelfile
cat > Modelfile << 'EOF'
FROM llama3.2:8b-instruct-q4_K_M
PARAMETER temperature 0.3
PARAMETER top_p 0.95
PARAMETER num_ctx 8192
TEMPLATE """{{ if .System }}<|start_header_id|>system<|end_header_id|>

{{ .System }}<|eot_id|>{{ end }}<|start_header_id|>user<|end_header_id|>

{{ .Prompt }}<|eot_id|><|start_header_id|>assistant<|end_header_id|>

"""
SYSTEM "You are a helpful coding assistant. Always provide working code with explanations."
EOF

ollama create my-coding-assistant -f Modelfile
ollama run my-coding-assistant "Write a Python function to find prime numbers"

# 3. Serve via vLLM with OpenAI-compatible API
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-3.1-8B \
    --tensor-parallel-size 1 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 8192 \
    --dtype bfloat16 \
    --port 8000

# 4. Query
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "meta-llama/Llama-3.1-8B",
        "messages": [{"role": "user", "content": "Write Python code for binary search"}],
        "temperature": 0.3,
        "max_tokens": 500
    }'
`

| Quantization | Model Size (7B) | Perplexity | Speed (tok/s) | GPU Memory |
|-------------|-----------------|------------|---------------|------------|
| fp16 | 14 GB | 5.12 | 45 | 16 GB |
| int8 (bitsandbytes) | 7 GB | 5.14 | 40 | 10 GB |
| q4_k_m (GGUF) | 4.1 GB | 5.28 | 35 | 6 GB |
| q4_0 (GGUF) | 3.9 GB | 5.35 | 38 | 5.5 GB |
| q3_k_m (GGUF) | 3.3 GB | 5.55 | 40 | 5 GB |
| q2_k (GGUF) | 2.7 GB | 6.12 | 42 | 4 GB |


---

## Expanded Content: Chapters 114-120

### Chapter 114: Function Calling — Complex Use Cases

`python
from openai import OpenAI
import json

client = OpenAI()

# Multi-tool schema with nested objects
tools = [
    {
        "type": "function",
        "function": {
            "name": "book_flight",
            "description": "Book a flight for a customer",
            "parameters": {
                "type": "object",
                "properties": {
                    "passenger": {
                        "type": "object",
                        "properties": {
                            "first_name": {"type": "string"},
                            "last_name": {"type": "string"},
                            "date_of_birth": {"type": "string", "format": "date"},
                        },
                        "required": ["first_name", "last_name"],
                    },
                    "flight": {
                        "type": "object",
                        "properties": {
                            "origin": {"type": "string", "description": "IATA airport code"},
                            "destination": {"type": "string", "description": "IATA airport code"},
                            "date": {"type": "string", "format": "date"},
                            "class": {"type": "string", "enum": ["economy", "business", "first"]},
                        },
                        "required": ["origin", "destination", "date"],
                    },
                    "payment": {
                        "type": "object",
                        "properties": {
                            "method": {"type": "string", "enum": ["credit_card", "debit_card", "paypal"]},
                            "currency": {"type": "string", "default": "USD"},
                        },
                    },
                },
                "required": ["passenger", "flight"],
            },
        }
    },
    {
        "type": "function",
        "function": {
            "name": "search_hotels",
            "description": "Search for available hotels in a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string"},
                    "check_in": {"type": "string", "format": "date"},
                    "check_out": {"type": "string", "format": "date"},
                    "guests": {"type": "integer", "minimum": 1, "maximum": 10},
                    "min_rating": {"type": "number", "minimum": 1, "maximum": 5},
                    "max_price": {"type": "number"},
                },
                "required": ["city", "check_in", "check_out"],
            },
        }
    }
]

# Conversation with function calling
def travel_agent():
    messages = [{"role": "system", "content": "You are a travel agent assistant. Help users book flights and hotels."}]
    messages.append({"role": "user", "content": "Book a flight from JFK to LHR on Dec 15 for John Doe in business class, and search for hotels in London from Dec 15-20"})

    response = client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
        tools=tools,
        tool_choice="auto",
    )

    for tool_call in response.choices[0].message.tool_calls:
        name = tool_call.function.name
        args = json.loads(tool_call.function.arguments)
        print(f"Calling {name} with:\n{json.dumps(args, indent=2)}")
        # Result would come from actual API calls

# Instructor for Pydantic validation
from pydantic import BaseModel, Field
from typing import List, Optional

class FlightBooking(BaseModel):
    passenger_name: str = Field(description="Full name of passenger")
    origin: str = Field(description="IATA origin airport code")
    destination: str = Field(description="IATA destination airport code")
    date: str = Field(description="Flight date in YYYY-MM-DD")
    class_of_service: str = Field(description="economy, business, or first")
    meal_preferences: Optional[List[str]] = None

    def validate_airport_codes(self):
        assert len(self.origin) == 3, "Origin must be IATA code"
        assert len(self.destination) == 3, "Destination must be IATA code"
`

### Chapter 115: AI Agents — Advanced Implementation

`python
import asyncio
from typing import List, Dict, Any
from dataclasses import dataclass, field

@dataclass
class AgentState:
    task: str
    completed_subtasks: List[str] = field(default_factory=list)
    current_step: int = 0
    max_steps: int = 15
    memory: Dict[str, Any] = field(default_factory=dict)
    tool_results: List[Dict] = field(default_factory=list)

class AdvancedAgent:
    def __init__(self, tools: Dict):
        self.tools = tools
        self.state = AgentState(task="")

    async def plan(self, task: str) -> List[str]:
        planning_prompt = f"""Decompose this task into sequential steps:
Task: {task}
Output format: numbered list of specific actions needed."""
        response = await llm.apredict(planning_prompt)
        steps = [s.strip() for s in response.split("\\n") if s.strip()]
        return steps

    async def execute_step(self, step: str) -> str:
        thought_prompt = f"""Current step: {step}
Previous context: {self.state.memory}
What tool should I use and what input should I give it?"""
        thought = await llm.apredict(thought_prompt)

        # Parse tool call from thought
        for tool_name, tool_fn in self.tools.items():
            if tool_name in thought:
                result = await tool_fn(thought)
                self.state.tool_results.append({"tool": tool_name, "result": result})
                return result
        return f"No tool needed for step: {step}"

    async def reflect(self, result: str) -> bool:
        reflection_prompt = f"""Task: {self.state.task}
Step result: {result}
Completed: {len(self.state.completed_subtasks)}/{self.state.max_steps}
Is the task complete? Answer YES or NO and explain why."""

        response = await llm.apredict(reflection_prompt)
        return "YES" in response.upper()

    async def run(self, task: str) -> str:
        self.state.task = task
        steps = await self.plan(task)

        for step in steps:
            if self.state.current_step >= self.state.max_steps:
                return "Max steps reached"
            result = await self.execute_step(step)
            self.state.completed_subtasks.append(step)
            self.state.current_step += 1
            self.state.memory[step] = result

            if await self.reflect(result):
                break

        # Final synthesis
        synthesis_prompt = f"""Task: {task}
Results: {self.state.memory}
Create a comprehensive final response."""
        return await llm.apredict(synthesis_prompt)
`

### Chapter 116: Multi-Agent — AutoGen Deep Dive

`python
from autogen import AssistantAgent, UserProxyAgent, GroupChat, GroupChatManager
from autogen.code_utils import create_virtual_env

# Create agents with specialized system messages
researcher = AssistantAgent(
    name="Researcher",
    system_message="""You are a researcher agent. Your job is to:
1. Search for information on the given topic
2. Summarize findings concisely
3. Provide citations and sources
4. Identify key facts and figures""",
    llm_config={"config_list": [{"model": "gpt-4o", "api_key": "..."}]},
)

coder = AssistantAgent(
    name="Coder",
    system_message="""You are a Python coding agent. Your job is to:
1. Write clean, well-documented Python code
2. Include unit tests
3. Handle edge cases
4. Follow PEP 8 style""",
    llm_config={"config_list": [{"model": "gpt-4o"}]},
)

reviewer = AssistantAgent(
    name="Reviewer",
    system_message="""You are a code reviewer. Your job is to:
1. Check for bugs and logic errors
2. Suggest performance improvements
3. Verify test coverage
4. Ensure code quality standards""",
    llm_config={"config_list": [{"model": "gpt-4o"}]},
)

# Create task-specific agents for complex workflows
planner = AssistantAgent(
    name="Planner",
    system_message="Decompose the task into clear, sequential steps. Assign each step to the appropriate agent.",
    llm_config={"config_list": [{"model": "gpt-4o"}]},
)

executor = UserProxyAgent(
    name="Executor",
    human_input_mode="NEVER",
    code_execution_config={
        "work_dir": "coding",
        "use_docker": "python:3.10-slim",
        "timeout": 60,
    },
)

# Group chat with manager
group_chat = GroupChat(
    agents=[planner, researcher, coder, reviewer, executor],
    messages=[],
    max_round=20,
    speaker_selection_method="auto",  # Can also be "round_robin" or "random"
)

manager = GroupChatManager(
    groupchat=group_chat,
    llm_config={"config_list": [{"model": "gpt-4o"}]},
)

# Start conversation
executor.initiate_chat(
    manager,
    message="Create a web scraper that extracts product prices from an e-commerce site, analyzes price trends, and generates a report with visualization.",
)
`

### Chapter 117: Agent Frameworks — Production Patterns

`python
# LangGraph: Supervisor with dynamic agent creation
from langgraph.graph import StateGraph, END, MessageGraph
from langgraph.checkpoint.sqlite import SqliteSaver
from typing import TypedDict, Literal

class WorkflowState(TypedDict):
    input: str
    plan: list
    current_step: int
    results: dict
    errors: list

def create_agent_graph():
    workflow = StateGraph(WorkflowState)

    def planner_node(state):
        response = llm.invoke(f"Plan steps for: {state['input']}")
        return {"plan": parse_steps(response)}

    def executor_node(state):
        step = state["plan"][state["current_step"]]
        try:
            result = execute_agent_step(step)
            return {"results": {step: result}, "current_step": state["current_step"] + 1}
        except Exception as e:
            return {"errors": [str(e)]}

    def router(state) -> Literal["executor", "finalizer", "error_handler"]:
        if state.get("errors"):
            return "error_handler"
        if state["current_step"] >= len(state["plan"]):
            return "finalizer"
        return "executor"

    workflow.add_node("planner", planner_node)
    workflow.add_node("executor", executor_node)
    workflow.add_node("error_handler", error_handler)
    workflow.add_node("finalizer", finalizer_node)

    workflow.set_entry_point("planner")
    workflow.add_edge("planner", "executor")
    workflow.add_conditional_edges("executor", router)
    workflow.add_edge("error_handler", "executor")
    workflow.add_edge("finalizer", END)

    return workflow.compile(checkpointer=SqliteSaver.from_conn_string("agents.db"))

# CrewAI: Hierarchical process
from crewai import Agent, Task, Crew, Process

manager = Agent(role="Project Manager", goal="Coordinate the team", allow_delegation=True)
analyst = Agent(role="Data Analyst", goal="Analyze data and find insights")
writer = Agent(role="Technical Writer", goal="Write clear documentation")

analysis = Task(description="Analyze the dataset", agent=analyst)
documentation = Task(description="Document the findings", agent=writer)

crew = Crew(
    agents=[manager, analyst, writer],
    tasks=[analysis, documentation],
    process=Process.hierarchical,
    manager_agent=manager,
)

# DSPy: Programmatic optimization
import dspy
from dspy.teleprompt import BootstrapFewShot

class ExtractInfo(dspy.Signature):
    \"\"\"Extract structured information from text.\"\"\"
    text = dspy.InputField()
    name = dspy.OutputField()
    age = dspy.OutputField()
    occupation = dspy.OutputField()

class InfoExtractor(dspy.Module):
    def __init__(self):
        self.extract = dspy.ChainOfThought(ExtractInfo)
    def forward(self, text):
        return self.extract(text=text)

# Compile with optimization
teleprompter = BootstrapFewShot(metric=lambda pred, gold: pred.name == gold.name)
optimized = teleprompter.compile(InfoExtractor(), trainset=dataset)
`

### Chapter 118: Safety — Production Implementation

`python
from fastapi import FastAPI, HTTPException, Request
from pydantic import BaseModel
import re
import hashlib
import time

app = FastAPI()

# Rate limiter
class RateLimiter:
    def __init__(self, max_requests=60, window=60):
        self.max_requests = max_requests
        self.window = window
        self.requests = {}

    def check(self, client_id: str) -> bool:
        now = time.time()
        if client_id not in self.requests:
            self.requests[client_id] = []
        self.requests[client_id] = [t for t in self.requests[client_id] if now - t < self.window]
        if len(self.requests[client_id]) >= self.max_requests:
            return False
        self.requests[client_id].append(now)
        return True

limiter = RateLimiter()

# Input sanitizer
class InputSanitizer:
    def __init__(self):
        self.blocked_patterns = [
            r"ignore all previous instructions",
            r"you are now",
            r"do anything now",
            r"system prompt",
            r"DAN",
        ]
        self.sensitive_patterns = [
            r"\b\d{3}-\d{2}-\d{4}\b",  # SSN
            r"\b\d{16}\b",  # Credit card
            r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}",  # Email
        ]

    def sanitize(self, text: str) -> str:
        for pattern in self.blocked_patterns:
            if re.search(pattern, text, re.IGNORECASE):
                return {"blocked": True, "reason": f"Pattern matched: {pattern}"}
        return {"blocked": False, "text": text}

    def detect_pii(self, text: str) -> list:
        found = []
        for pattern in self.sensitive_patterns:
            matches = re.findall(pattern, text)
            found.extend(matches)
        return found

sanitizer = InputSanitizer()

@app.post("/chat")
async def chat(request: Request, prompt: str):
    # 1. Rate limit
    client_ip = request.client.host
    if not limiter.check(client_ip):
        raise HTTPException(status_code=429, detail="Rate limit exceeded")

    # 2. Input sanitization
    sanitized = sanitizer.sanitize(prompt)
    if sanitized.get("blocked"):
        raise HTTPException(status_code=400, detail="Input blocked")

    # 3. Process with LLM
    response = llm.invoke(prompt)

    # 4. Output filtering
    pii = sanitizer.detect_pii(response)
    if pii:
        response = redact_pii(response, pii)

    # 5. Log for audit
    log_interaction(client_ip, prompt, response)

    return {"response": response}
`

### Chapter 119: Evaluation — Comprehensive Suite

`python
import evaluate
from ragas import evaluate as ragas_evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall
from datasets import Dataset

class EvalSuite:
    def __init__(self):
        self.bleu = evaluate.load("bleu")
        self.rouge = evaluate.load("rouge")
        self.bertscore = evaluate.load("bertscore")
        self.meteor = evaluate.load("meteor")

    def compute_extrinsic(self, predictions, references):
        return {
            "bleu": self.bleu.compute(predictions=predictions, references=references),
            "rouge": self.rouge.compute(predictions=predictions, references=references),
            "meteor": self.meteor.compute(predictions=predictions, references=references),
            "bertscore": self.bertscore.compute(
                predictions=predictions, references=references, lang="en"
            ),
        }

    def compute_ragas(self, questions, answers, contexts, ground_truths):
        dataset = Dataset.from_dict({
            "question": questions,
            "answer": answers,
            "contexts": contexts,
            "ground_truth": ground_truths,
        })
        return ragas_evaluate(
            dataset=dataset,
            metrics=[faithfulness, answer_relevancy, context_precision, context_recall],
        )

    def llm_as_judge(self, question, answer, rubric="1-5"):
        prompt = f"""Evaluate this answer:
Question: {question}
Answer: {answer}

Score {rubric} based on: accuracy, completeness, clarity.
Provide score and justification."""
        return llm.invoke(prompt)

eval_suite = EvalSuite()

# Example usage
predictions = ["The capital of France is Paris."]
references = [["Paris is the capital city of France."]]
extrinsic = eval_suite.compute_extrinsic(predictions, references)
print(f"BLEU: {extrinsic['bleu']['bleu']:.3f}, BERTScore F1: {extrinsic['bertscore']['f1'][0]:.3f}")
`

### Chapter 120: Production — Architecture Implementation

`python
import asyncio
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
from typing import Optional
import redis.asyncio as redis
import json

app = FastAPI()

# Redis for caching
cache = redis.Redis(host="localhost", port=6379, decode_responses=True)

class QueryRequest(BaseModel):
    query: str
    user_id: str
    model: Optional[str] = "auto"

class QueryResponse(BaseModel):
    answer: str
    sources: list
    model_used: str
    latency_ms: float

# Model routing
MODEL_ROUTES = {
    "simple": "llama-3.2-3b",
    "medium": "llama-3.1-8b",
    "complex": "llama-3.1-70b",
}

def classify_query(query: str) -> str:
    if len(query) < 50 and "?" in query:
        return "simple"
    elif any(word in query for word in ["explain", "compare", "analyze", "why", "how"]):
        return "complex"
    return "medium"

@app.post("/query", response_model=QueryResponse)
async def query_endpoint(request: QueryRequest, background_tasks: BackgroundTasks):
    import time
    start = time.time()

    # 1. Check cache
    cache_key = f"query:{hash(request.query)}"
    cached = await cache.get(cache_key)
    if cached:
        result = json.loads(cached)
        return QueryResponse(**result, latency_ms=(time.time() - start) * 1000)

    # 2. Model routing
    if request.model == "auto":
        complexity = classify_query(request.query)
        model = MODEL_ROUTES[complexity]
    else:
        model = request.model

    # 3. Retrieve context
    query_emb = embedding_model.encode(request.query)
    docs = vector_db.search(query_emb, k=5)
    reranked = cross_encoder.rerank(request.query, docs)[:3]
    context = "\\n\\n".join([d.text for d in reranked])

    # 4. Generate
    prompt = f"Context: {context}\\n\\nQuestion: {request.query}\\n\\nAnswer:"
    response = llm_generate(model, prompt)

    # 5. Cache result
    result = {
        "answer": response,
        "sources": [d.metadata["source"] for d in reranked],
        "model_used": model,
    }
    background_tasks.add_task(cache.setex, cache_key, 3600, json.dumps(result))

    # 6. Log for monitoring
    background_tasks.add_task(log_metrics, {
        "user_id": request.user_id,
        "model": model,
        "latency": (time.time() - start) * 1000,
        "tokens": count_tokens(response),
        "cost": calculate_cost(model, count_tokens(response)),
    })

    return QueryResponse(**result, latency_ms=(time.time() - start) * 1000)

# Kubernetes-ready health check
@app.get("/health")
async def health():
    return {
        "status": "healthy",
        "cache_connected": await cache.ping(),
        "model_loaded": model.is_loaded(),
        "queue_depth": get_queue_depth(),
    }
`


---

## Expanded Content: Architecture Diagrams & Detailed Explanations

### Complete RAG Architecture with All Components

`mermaid
graph TB
    subgraph "Data Ingestion"
        PDF[PDF Documents] --> PARSER[Document Parser<br/>PyMuPDF / Unstructured / OCR]
        HTML[HTML Pages] --> PARSER
        CODE[Code Repos] --> PARSER
        DB[(Databases)] --> PARSER
        PARSER --> CHUNK[Chunking Strategies]
        CHUNK --> RECURSIVE[RecursiveCharacter<br/>chunk_size=512, overlap=50]
        CHUNK --> SEMANTIC[Semantic Chunking<br/>Sentence boundaries]
        CHUNK --> STRUCTURED[Structure-aware<br/>Markdown / Code headers]
    end

    subgraph "Embedding & Indexing"
        RECURSIVE --> EMBED[Embedding Models]
        SEMANTIC --> EMBED
        STRUCTURED --> EMBED
        EMBED --> BGE[BGE-large-en-v1.5<br/>1024-dim]
        EMBED --> ADA[text-embedding-3-small<br/>1536-dim]
        EMBED --> MINI[all-MiniLM-L6-v2<br/>384-dim]
        BGE --> STORE[(Vector Database)]
        ADA --> STORE
        MINI --> STORE
        STORE --> INDEX[ANN Index<br/>HNSW: M=32, efC=200]
    end

    subgraph "Query Processing"
        Q[User Query] --> ROUTE{Query Classifier}
        ROUTE -->|Simple| SIMPLE[Direct Retrieval<br/>k=3]
        ROUTE -->|Complex| COMPLEX[Query Transformation]
        COMPLEX --> REWRITE[Query Rewriting<br/>LLM-based reformulation]
        COMPLEX --> HYDE[HyDE<br/>Hypothetical Document]
        COMPLEX --> MULTI[Multi-Query<br/>5 variations]
        REWRITE --> QEMB[Query Embedding]
        HYDE --> QEMB
        MULTI --> QEMB
    end

    subgraph "Retrieval & Fusion"
        QEMB --> SEARCH1[Dense Search<br/>Vector similarity]
        QEMB --> SEARCH2[Sparse Search<br/>BM25 keyword]
        SEARCH1 --> FUSE[Fusion<br/>RRF / DBSF]
        SEARCH2 --> FUSE
        FUSE --> TOPK[Top-k Candidates<br/>k=20]
        TOPK --> RERANK[Cross-encoder Re-ranking<br/>BGE-reranker-v2-m3]
    end

    subgraph "Generation"
        RERANK --> CONTEXT[Context Assembly<br/>Top-5 after reranking]
        CONTEXT --> PROMPT[Prompt Construction<br/>System + Context + Query]
        PROMPT --> LLM[LLM Generation<br/>GPT-4o / LLaMA 3]
        LLM --> CHECK{Quality Check}
        CHECK -->|Pass| RESPONSE[Final Response<br/>with Citations]
        CHECK -->|Fail - Low Confidence| RETRY[Retry with<br/>expanded context]
        RETRY --> CONTEXT
        CHECK -->|Fail - Hallucination| CORRECT[Corrective RAG<br/>Self-reflection]
        CORRECT --> CONTEXT
    end
`

### LLM Training Pipeline Architecture

`mermaid
graph TB
    subgraph "Data Preparation"
        WEB[Web Crawl<br/>CommonCrawl] --> FILTER[Quality Filtering<br/>Deduplication / PII / Toxicity]
        BOOKS[Books / Papers] --> FILTER
        CODE[Code Repos] --> FILTER
        WIKI[Wikipedia] --> FILTER
        FILTER --> TOKENIZE[Tokenization<br/>BPE / SentencePiece]
        TOKENIZE --> SHARDS[Data Shards<br/>~10GB each]
    end

    subgraph "Pre-training"
        SHARDS --> TRAIN[Training Loop]
        TRAIN --> MODEL[Model<br/>GPT / LLaMA / MoE]
        TRAIN --> OPT[Optimizer<br/>AdamW / Lion]
        TRAIN --> LR[LR Schedule<br/>Cosine / Warmup]
        TRAIN --> LOSS[Loss<br/>Cross-entropy / Auxiliary]
        MODEL --> FSDP[FSDP / ZeRO-3<br/>Sharded Training]
        FSDP --> TP[Tensor Parallel<br/>Layer splitting]
        TP --> PP[Pipeline Parallel<br/>Stage splitting]
    end

    subgraph "Post-training"
        MODEL --> SFT[Supervised Fine-Tuning<br/>Instruction datasets]
        SFT --> RM[Reward Model<br/>Bradley-Terry / Pairwise]
        RM --> PPO[PPO Training<br/>RLHF]
        PPO --> ALIGN[Aligned Model]
        SFT --> DPO[Direct Preference Optimization]
        DPO --> ALIGN
    end

    subgraph "Evaluation"
        ALIGN --> BENCH[Benchmarks<br/>MMLU / GSM8K / HumanEval]
        ALIGN --> SAFETY[Safety Eval<br/>Red teaming / Toxicity]
        ALIGN --> HUMAN[Human Eval<br/>Chatbot Arena]
    end
`

### Inference Serving Architecture with Scaling

`mermaid
graph TB
    subgraph "Client Layer"
        WEB[Web App] --> CDN[CDN / CloudFront]
        MOBILE[Mobile App] --> CDN
        API[External API] --> WAF[WAF]
    end

    subgraph "Gateway Layer"
        CDN --> ALB[Load Balancer<br/>Round Robin / Least Connections]
        WAF --> ALB
        ALB --> API_GW[API Gateway<br/>Kong / Envoy]
        API_GW --> AUTH[Auth Service<br/>JWT / API Keys]
        API_GW --> RATE[Rate Limiter<br/>Token Bucket / Sliding Window]
    end

    subgraph "Application Layer"
        AUTH --> APP[App Server<br/>FastAPI / Django]
        RATE --> APP
        APP --> CACHE1[Response Cache<br/>Redis Cluster]
        APP --> ROUTER[Model Router<br/>3B / 8B / 70B]
    end

    subgraph "LLM Serving Layer"
        ROUTER --> QUEUE[Request Queue<br/>RabbitMQ / Kafka]
        QUEUE --> VLLM1[vLLM Instance 1<br/>PagedAttention]
        QUEUE --> VLLM2[vLLM Instance 2<br/>Continuous Batching]
        QUEUE --> VLLMN[vLLM Instance N<br/>Auto-scaled]
        VLLM1 --> GPU_POOL[GPU Pool<br/>A100 / H100]
        VLLM2 --> GPU_POOL
        VLLMN --> GPU_POOL
    end

    subgraph "RAG Layer"
        VLLM1 --> RAG[RAG Service]
        RAG --> EMBED_SVC[Embedding Service<br/>Batch / Async]
        RAG --> RETRIEVER[Retriever<br/>Hybrid Dense + Sparse]
        RETRIEVER --> VDB[(Vector DB<br/>Qdrant / Milvus)]
        RETRIEVER --> BM25[(BM25 Index<br/>Elasticsearch)]
        RAG --> RERANKER[Re-ranker<br/>Cross-encoder Service]
    end

    subgraph "Observability"
        ALL_COMPONENTS --> PROM[Prometheus<br/>Metrics Collection]
        PROM --> GRAF[Grafana<br/>Dashboards / Alerts]
        ALL_COMPONENTS --> TRACE[Distributed Tracing<br/>OpenTelemetry / Jaeger]
        ALL_COMPONENTS --> LOGS[Log Aggregation<br/>ELK / Loki]
    end
`

### Production Cost Breakdown (per 1M queries)

| Component | Model | Cost/1M Queries | % of Total |
|-----------|-------|-----------------|------------|
| LLM Inference | GPT-4o (50% queries) | ,000 | 45% |
| LLM Inference | GPT-4o-mini (50% queries) |  | 7% |
| Embedding | text-embedding-3-small |  | 2% |
| Vector DB | Qdrant Cloud (10M vectors) | ,500 | 14% |
| Re-ranking | BGE-reranker (GPU) |  | 5% |
| Serving Infra | 4x A100 GPUs | ,200 | 29% |
| Caching (Redis) | Cache tier |  | 2% |
| Monitoring | Prometheus + Grafana |  | 1% |
| **Total** | | **,610** | **100%** |

### Latency Budget (target < 2s p95)

| Component | Budget | Optimization |
|-----------|--------|-------------|
| Network (client -> server) | 100ms | CDN, edge computing |
| Auth + Rate limiting | 20ms | In-memory token store |
| Query classification | 50ms | Lightweight classifier (distilbert) |
| Query embedding | 30ms | Pre-computed for common queries |
| ANN search (10M vectors) | 50ms | HNSW with optimized efSearch |
| BM25 search | 20ms | Elasticsearch |
| Re-ranking (20 -> 5) | 150ms | BGE-reranker, batch inference |
| Context assembly | 10ms | Pre-formatted templates |
| LLM generation (200 tok) | 500ms | vLLM with continuous batching |
| Output filtering | 20ms | Regex + classifier (100ms parallel) |
| Response serialization | 50ms | Optimized JSON |
| **Total** | **~1000ms** | |

### Prompt Template Library

`python
TEMPLATES = {
    "qa": {
        "system": "You are a helpful assistant. Answer based on the provided context.",
        "template": """Context:
{context}

Question: {question}

Answer concisely and cite sources from the context:""",
    },
    "code_review": {
        "system": "You are a senior software engineer reviewing code.",
        "template": """Review the following {language} code for:
1. Bug and logic errors
2. Security vulnerabilities
3. Performance issues
4. Code style (follow {style_guide})

`{language}
{code}
`

Provide severity ratings: CRITICAL, HIGH, MEDIUM, LOW for each issue found.""",
    },
    "analysis": {
        "system": "You are a data analyst. Provide structured analysis.",
        "template": """Analyze the following {data_type}:

{data}

Output format:
- Summary: (2-3 sentences)
- Key Insights: (numbered list)
- Recommendations: (numbered with priority)
- Confidence Level: (HIGH/MEDIUM/LOW)
- Data Gaps: (what's missing)""",
    },
    "chat": {
        "system": "You are a helpful {persona}. Maintain conversation history.",
        "template": """Previous conversation:
{history}

User: {message}
Assistant:""",
    },
}
`

