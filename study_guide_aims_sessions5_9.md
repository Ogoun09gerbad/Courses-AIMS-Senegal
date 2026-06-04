# COMPLETE STUDY GUIDE
## GAAI-AIMS — Module 2 & Module 3 (Sessions 5 to 9)
### Prompt Engineering · RAG · Fine-Tuning · Evaluation · LLM Agents

*Based strictly on the course slides by Dr. Papa-Séga WADE, AIMS Senegal*

---

# I — COURSE SUMMARIES

---

## I-1 — cours_aims_module2_session5
### Prompt Engineering & Context Management

---

### I-1-a — Summary and Important Points to Remember

#### Course Objective

By the end of Session 5, you should be able to **write prompts that give reliable, reproducible answers from an LLM**. You will know how to ask the model to reason step by step, how to force a specific output format, and when to stop using prompting and switch to a different tool.

The guiding question of the session is: *"For each AIMS task, what is the simplest prompt that gives a reliable, reproducible answer?"*

---

#### Main Concepts

**1. What is a Prompt?**

A prompt is NOT just a question you type into a chat box. According to the course: **"The prompt is not a question. It is a specification of the desired output."**

Think of it like writing instructions for a new employee. If you say "tell me about attention," the employee could talk about anything — psychology, marketing, or neural networks. But if you say "You are an NLP instructor. In 5 bullets, explain self-attention to a Master 2 student who knows linear algebra. Use one analogy." — you get exactly what you need.

---

#### Key Techniques

**2. The Five Parts of a Good Prompt (Anatomy)**

Every well-designed prompt has five building blocks:

| Part | What it does | Example |
|---|---|---|
| **Role** | Tells the model who to be | "You are a senior NLP researcher" |
| **Task** | Tells the model what to produce | "Review this thesis" |
| **Context** | Gives facts the model needs | "The document is about RAG systems" |
| **Format** | Defines the shape of the answer | "Return a JSON object with these keys: …" |
| **Constraints** | Limits length, language, what NOT to do | "Max 150 words. No speculation." |

---

**3. Zero-Shot Prompting**

You give NO examples. You just write the instruction and the model has to figure it out.

- **When it works well:** Simple tasks like translation, summarization, or basic classification.
- **When it fails:** Rare formats, domain-specific tasks, or when the model must follow a very precise structure.

*Example:*
```
Translate the following sentence from English to French:
"The course starts on May 18."
```

---

**4. Few-Shot Prompting**

You give 2–5 **examples** of input/output before asking your real question. The model learns the pattern from the examples and imitates it.

- **Why it helps:** It removes ambiguity, teaches the exact output format, and helps smaller models that struggle with instructions.
- **Practical rule:** 3 to 5 examples is usually enough. Too many waste context.

*Example:*
```
Extract the country of birth.
Text: "Marie was born in Dakar."
Country: Senegal
Text: "Ahmed grew up in Cairo."
Country: Egypt
Text: "Lina was born in Bamako."
Country:
```

---

**5. Role Prompting**

You give the model a **persona** (a character to play). This changes the style, vocabulary, and depth of the answer.

- "teacher" → explains step by step
- "reviewer" → critical, structured
- "translator" → faithful, terminologically correct

**Important caution:** Roles change tone and depth, NOT factual accuracy. A role does NOT fix hallucinations (invented facts).

---

**6. Format Constraints: JSON, Tables, Bullets**

When your application needs to process the model's answer with code, you MUST enforce a structured format. Unstructured prose is hard to parse.

*JSON example prompt ending:*
```
Return a JSON object with keys:
{ "summary": str (max 3 sentences), "topics": list[str], "language": "en" | "fr" }
No prose outside the JSON.
```

**Engineering tip:** End constrained prompts with negative instructions: "no preamble", "no explanation", "no markdown fences."

---

**7. Chain-of-Thought (CoT)**

Instead of asking the model to jump straight to the answer, you ask it to **reason step by step first**. This dramatically improves results on math, logic, and multi-step tasks.

*Two versions:*
- **Zero-shot CoT:** Just add "Let's think step by step." to your prompt.
- **Few-shot CoT:** Provide examples where the reasoning is shown before the answer.

*When NOT to use CoT:* Trivial questions, simple format tasks, or when speed matters. CoT uses more tokens and costs more.

---

**8. Self-Consistency**

Generate the **same prompt k times** (e.g. k=5) with some randomness (temperature > 0), then take the **majority vote** among the answers.

- Best for: math word problems, logical reasoning, multi-step extraction.
- Cost: k samples = k× the cost. The course recommends k ∈ {3, 5, 7}.
- Self-consistency typically lifts accuracy by 5 to 15 points on arithmetic word problems.

---

**9. Quick Comparison Table of Techniques**

| Technique | Best for | Token cost | Reliability |
|---|---|---|---|
| Zero-shot | Common tasks | Low | Medium |
| Few-shot | Format / rare tasks | Medium | Good |
| Role | Tone, depth, audience | Low | Medium |
| CoT | Math, logic, multi-hop | High | Good |
| Self-consistency | High-stakes reasoning | Very high | Best |

**Heuristic:** Start with the cheapest reliable technique. Escalate only when the task fails reproducibly.

---

**10. The Context Window**

The context window is the **maximum number of tokens** (words/pieces) the model can see at once — both your input AND its output combined.

- Small models: 8K to 32K tokens
- Modern open models: 128K
- Frontier models: 1M+ (Gemini, Claude long-context)

**Important:** Long context ≠ free. You pay per token, and the model becomes slower with longer inputs.

---

**11. The "Lost in the Middle" Effect**

A research finding (Liu et al., 2023): when important information is **placed in the middle** of a long context, models retrieve it less reliably than when it is at the **start or end**.

**Practical rule:**
- Put critical instructions **first**
- Put critical evidence **last**
- Never assume the middle is safe

---

**12. Structuring a Long Prompt**

The recommended layout for long inputs:
1. System role and task (top)
2. Output format and constraints
3. Optional few-shot examples
4. Reference documents (clearly delimited)
5. Final instruction **restating the task** (at the bottom)

**Why repeat the task at the end?** Restating the instruction after the documents helps the model remember what it is supposed to do — this counters the lost-in-the-middle effect.

Use clear delimiters: `### DOCUMENT 1 ###`, `<doc>...</doc>`, or triple backticks. Be consistent.

---

**13. When Prompting Is Not Enough**

Prompting has three hard limits:

| Limit | What it means |
|---|---|
| **Hallucinations** | The model generates plausible but false content with full confidence |
| **Knowledge cutoff** | The model doesn't know events or papers after its training date |
| **Context window** | Even 1M tokens is finite; large corpora don't fit |

**Conclusion:** Prompting cannot give the model new factual knowledge or make permanent behavior changes. This is why the course introduces RAG (Session 6) and Fine-Tuning (Session 7).

**Decision tree to remember:**
- If prompting is enough → use prompting
- If you need specific knowledge → RAG
- If you need to change style or behavior → fine-tuning

---

#### Important Examples from the Course

**Vague vs. Engineered prompt:**
- Vague: "Tell me about attention." → Generic paragraph, could be about psychology or marketing. Not reproducible.
- Engineered: "You are an NLP instructor. In 5 bullets, explain self-attention to a Master 2 student who knows linear algebra. Use one analogy." → Targeted, structured, reproducible.

**AIMS Lab Tasks (4 exercises):**
1. Summarize a scientific abstract in 3 sentences
2. Explain a math concept (e.g. softmax) to a Master 1 student
3. Generate Python code: read a CSV and plot a histogram
4. Extract structured info from French text into JSON

---

#### Final Takeaways

1. A prompt is a specification of the desired output, not just a question.
2. The anatomy is: role, task, context, format, constraints.
3. Escalate techniques only as needed: zero-shot → few-shot → CoT → self-consistency.
4. Place critical information at the edges of long prompts.
5. Prompting cannot fix missing knowledge or stale facts. RAG and fine-tuning will.

---

### I-1-b — Quiz Questions and Answers

**Q1.** What is a prompt, according to Session 5?
**Answer:** A prompt is not a question — it is a specification of the desired output. It tells the model exactly what to produce, in what format, with what constraints.

**Q2.** What are the five parts of a well-structured prompt?
**Answer:** Role (who the model should be), Task (what to produce), Context (facts the model needs), Format (shape of the output), and Constraints (length, language, what NOT to do).

**Q3.** What is Zero-Shot prompting? Give an example of when it works well.
**Answer:** Zero-shot prompting means giving no examples — just an instruction. It works well for common tasks like translation, summarization, or classification, when the output format is simple and the instruction is unambiguous.

**Q4.** How many examples are recommended for Few-Shot prompting, and why not more?
**Answer:** 3 to 5 examples is usually enough. Too many examples waste context window space without meaningfully improving performance.

**Q5.** What is a "role" in Role Prompting, and what does it NOT fix?
**Answer:** A role is a persona given to the model (e.g. "teacher", "reviewer", "translator") that changes its tone, vocabulary, and depth. However, a role does NOT fix hallucinations — it does not make the model more factually accurate.

**Q6.** Why is it important to enforce structured output formats like JSON in a prompt?
**Answer:** Structured output can be directly parsed by code. Unstructured prose requires extra processing steps (regex, retries, additional LLM calls), which is inefficient and error-prone.

**Q7.** What does Chain-of-Thought (CoT) prompting do? What are the two versions?
**Answer:** CoT asks the model to reason step by step before giving the final answer. The two versions are: (1) Zero-shot CoT — add "Let's think step by step." to the prompt; and (2) Few-shot CoT — provide examples that include the full reasoning before the answer.

**Q8.** When should you NOT use Chain-of-Thought?
**Answer:** Avoid CoT for trivial questions, format-only tasks, and latency-critical applications, because CoT increases the number of tokens used and therefore cost and response time.

**Q9.** Explain Self-Consistency in simple terms.
**Answer:** Self-consistency means running the same prompt multiple times with some randomness, then taking the most common answer (majority vote). It reduces the risk of getting an unusual or wrong reasoning chain. The course recommends k ∈ {3, 5, 7} samples.

**Q10.** What is the "Lost in the Middle" effect? What is the practical rule to avoid it?
**Answer:** Research by Liu et al. (2023) found that models retrieve information less reliably when it is placed in the middle of a long context. The practical rule is: put critical instructions first and critical evidence last; never assume the middle is safe.

**Q11.** What are the three hard limits of prompting?
**Answer:** (1) Hallucinations — the model invents plausible but false content; (2) Knowledge cutoff — the model doesn't know information after its training date; (3) Context window — even very long contexts have a finite limit, and large growing corpora don't fit.

**Q12.** You are building a system that must extract structured data from French text. Which prompting technique(s) would you combine?
**Answer:** Few-shot prompting (to show the exact JSON format expected) combined with format constraints (explicitly asking for JSON only, with negative instructions like "no prose outside the JSON").

**Q13.** What is the recommended layout for a long prompt with reference documents?
**Answer:** (1) System role and task at the top; (2) Output format and constraints; (3) Optional few-shot examples; (4) Reference documents clearly delimited; (5) Final instruction **restating the task** at the very end.

**Q14.** Why should you restate the task at the end of a long prompt?
**Answer:** Restating the instruction after the documents helps counteract the lost-in-the-middle effect — it re-anchors the model on what it is supposed to do after processing a large amount of reference text.

**Q15.** According to the module decision tree, when should you use fine-tuning instead of prompting?
**Answer:** You should use fine-tuning when you need to change the model's style, tone, or behavior permanently — not just for one query. Prompting is used when it's enough; RAG is used when the model needs new factual knowledge; fine-tuning is used when behavior itself must change.

**Q16.** In Self-Consistency, why shouldn't you use k > 7?
**Answer:** Beyond k = 7, accuracy gains plateau while cost continues to multiply. The course recommends k ∈ {3, 5, 7} as a practical trade-off between reliability and token cost.

**Q17.** What is the difference in token cost between Zero-Shot and Self-Consistency?
**Answer:** Zero-shot has low token cost (one single call). Self-consistency has very high token cost because it makes k independent calls (k× the cost of one call), then aggregates results via majority vote.

---

## I-2 — cours_aims_module2_session6
### Retrieval-Augmented Generation (RAG)

---

### I-2-a — Summary and Important Points to Remember

#### Course Objective

By the end of Session 6, you will be able to build a complete system that lets an LLM answer questions **based on specific documents you provide** — not based on what it memorized during training.

The guiding question: *"How would you build an assistant that answers students' questions about the AIMS Senegal academic regulations using only the official PDF?"*

---

#### What Problem Does RAG Solve?

**The core problem:** LLMs don't know YOUR documents.

Imagine asking an LLM: "What is the minimum grade to pass the GAAI module at AIMS Senegal this year?" The LLM either invents a plausible answer (hallucination) or refuses — because the AIMS-Senegal 2025-2026 regulations were never in its training data.

**Why not just paste the whole document into the prompt?** Three real limits:
1. **Size:** A 300-page handbook does not fit in 8K–32K tokens.
2. **Cost:** Even with 1M-token windows, paying for the full document on every query is wasteful.
3. **Noise:** More irrelevant text = more lost-in-the-middle effect.

**The RAG solution:** Find the few relevant passages first, then ask the LLM to answer using only those.

---

#### The Complete RAG Pipeline

RAG has **two distinct phases**:

**Phase 1 — Indexing (done once per document):**
Document → Split into Chunks → Create Embeddings → Store in Vector Database

**Phase 2 — Querying (done for every user question):**
User Question → Embed the Question → Search Vector Database → Retrieve Top-K Chunks → Build Prompt → LLM generates Answer

**Mental model:** RAG = a librarian (the retriever) hands the LLM the right pages, then the LLM reads and answers.

---

#### The Five Building Blocks

| Block | Role |
|---|---|
| **Loader** | Reads the PDF, HTML, or DOCX |
| **Chunker** | Splits the document into passages |
| **Embedder** | Converts text into vectors (numbers) |
| **Vector Store** | Stores and searches the vectors fast |
| **Retriever** | Finds the most relevant chunks for a question |
| **LLM** | Reads the chunks and produces the answer |
| *Reranker (optional)* | Refines the top-k chunks before sending to LLM |

---

#### Embeddings — Explained Simply

An **embedding** is like a GPS coordinate for a piece of text. Instead of physical location, it represents the *meaning* of the text in a multi-dimensional space (typically 384, 768, or 1024 dimensions).

**The key property:** Texts with similar meaning produce vectors that are **close together**, even if they use completely different words.

*Example from the course:*
- Query: "Où puis-je faire vacciner mon enfant à Dakar?"
- Document D2: "Les centres de santé de Pikine offrent la vaccination gratuite aux enfants."
- Even though the query says "Dakar" and D2 says "Pikine", the embedding model correctly ranks D2 first because the **meanings** are close.

**Cosine similarity** is used to measure how close two vectors are. A value close to 1 = very similar meaning.

**Models recommended in the course:**
- `all-MiniLM-L6-v2` — small, fast, English only
- `paraphrase-multilingual-MiniLM-L12-v2` — works with French and Wolof (important for AIMS context)
- `BGE-M3` — strong multilingual, longer context

---

#### Chunking — Why It Matters

**Chunking** means cutting a long document into smaller retrievable pieces.

**Two failure modes:**
- Chunks too **small** → lose context → no usable answer
- Chunks too **big** → dilute relevance → retrieval ranks them lower

**Practical defaults from the course:**
- Size: 300 to 800 tokens
- Overlap: 50 to 100 tokens (overlap stitches concepts split across chunks)
- Respect natural boundaries: paragraphs, headings, sentences

**Chunking strategies:**

| Strategy | How it works | When to use |
|---|---|---|
| Fixed-size | Cut every N tokens, fixed overlap | Simple generic baseline |
| Recursive | Split on paragraph breaks, then line breaks, then sentences | Narrative documents, AIMS handbooks |
| Semantic | Cut where meaning changes | Long structured reports |
| Document-aware | Respect headings, sections, table rows | Regulations, laws, thesis chapters |

**Heuristic:** Start with recursive splitting at 500 tokens / 80 overlap. Tune only if retrieval evaluation shows a problem.

---

#### Vector Databases

A **vector store** stores (vector, text, metadata) tuples and answers nearest-neighbor queries fast.

**Chroma** is recommended for AIMS labs because:
- Open source, pure Python install
- Embedded mode (no separate server needed)
- Works on Colab, laptop, offline
- Default distance metric: cosine similarity

**Production alternatives:** Qdrant, Weaviate, Milvus, pgvector — same concept, better for large-scale deployment.

---

#### Retrieval — Dense vs Sparse Search

| Type | How it works | Strength | Weakness |
|---|---|---|---|
| **Dense** (embeddings) | Meaning-based, uses vectors | Handles paraphrase and synonyms | Weak on rare proper nouns and codes |
| **Sparse** (BM25, TF-IDF) | Lexical overlap of exact words | Perfect on exact codes and IDs | Misses paraphrases |

*Example: "thiof" (local fish name) matches "poisson" (French for fish) with dense search but NOT with sparse search.*

**Hybrid retrieval:** Production systems often combine both: dense for meaning, sparse for codes and acronyms (like ANSD, DGID, CFA). Mentioned for awareness in the course.

---

#### Reranking (Two-Stage Retrieval)

For better results, use two stages:
1. **Bi-encoder** retrieves top-50 fast (cheap embeddings)
2. **Cross-encoder** reranks the 50 into top-5 (slow, but accurate — reads query AND chunk together)

Why two stages? Cross-encoders are too slow to apply to millions of chunks, but perfect to refine 50 candidates. This typically lifts answer quality by 5 to 15 points, at roughly 3× retrieval latency.

---

#### Common Failure Modes

| Failure | Cause | Fix |
|---|---|---|
| **Bad chunks** | Tables and equations mangled by PDF extraction | Inspect chunks before embedding |
| **Wrong embedder language** | English-only model on French/Wolof text | Always pick multilingual for AIMS corpora |
| **No-evidence answers** | LLM hallucinates when context is empty | Add explicit refusal rule: "If the context is silent, say I don't know" |
| **Stale index** | Documents updated but embeddings not re-generated | Re-embed when documents change |

---

#### Important Examples from the Course

**Senegalese health FAQ example:**
- Query: "Où puis-je faire vacciner mon enfant à Dakar?"
- The model correctly retrieves the sentence about "centres de santé de Pikine" offering free vaccination for children — even though "Pikine" ≠ "Dakar" — because the semantic meaning matches.

**African Use Cases Where RAG Shines:**
- AIMS academic assistant on internal regulations
- DGID (Senegalese tax) question assistant
- ANSD assistant for journalists querying census tables
- Rural-clinic chatbot citing official vaccination guidelines
- Wolof voice front-end + RAG over agriculture PDFs

---

#### Final Takeaways

1. RAG = retrieve relevant chunks, then let the LLM answer with them.
2. Indexing happens **once**; querying happens **every call**.
3. Embeddings encode **meaning**, not words. Sparse search complements them for exact codes.
4. Chunk size and overlap matter more than the choice of LLM.
5. Always force the model to refuse when the context is silent.
6. For AIMS/African contexts: pick **multilingual** embedders.

---

### I-2-b — Quiz Questions and Answers

**Q1.** What is the core problem that RAG solves?
**Answer:** LLMs don't know your specific documents. They can't answer questions about internal regulations, recent PDFs, or private data because that information was never in their training data. RAG solves this by retrieving relevant passages at query time.

**Q2.** Why can't you just paste the whole document into the prompt?
**Answer:** Three reasons: (1) Size — a 300-page document doesn't fit in 8K–32K tokens; (2) Cost — paying for the full document on every query is wasteful even with large context windows; (3) Noise — more irrelevant text causes the lost-in-the-middle effect, degrading answer quality.

**Q3.** What are the two phases of a RAG pipeline? Which happens once and which happens for every query?
**Answer:** (1) **Indexing** — splitting, embedding, and storing documents. This happens **once** per document. (2) **Querying** — embedding the user's question, searching the vector store, retrieving top-k chunks, and generating the answer. This happens **for every user question**.

**Q4.** What is an embedding in simple terms?
**Answer:** An embedding is a set of numbers (a vector) that represents the *meaning* of a piece of text. Texts with similar meanings get vectors that are close together in space. This allows the system to find similar content even when different words are used.

**Q5.** What does cosine similarity measure in the context of RAG?
**Answer:** Cosine similarity measures how close two embedding vectors are — which corresponds to how similar their meanings are. A value close to 1 means very similar meaning; a value close to 0 means unrelated.

**Q6.** Why is the multilingual model `paraphrase-multilingual-MiniLM-L12-v2` recommended for AIMS projects?
**Answer:** Because AIMS projects often involve French and sometimes Wolof text. English-only embedding models perform poorly on non-English text. A multilingual model produces meaningful embeddings for French and other African languages.

**Q7.** What are the two failure modes of chunking?
**Answer:** Chunks too **small** → lose context, no usable answer in the retrieved chunk. Chunks too **big** → dilute relevance, retrieval ranks them lower. The recommended size is 300–800 tokens with 50–100 token overlap.

**Q8.** What is chunk overlap and why is it used?
**Answer:** Overlap means the end of one chunk and the beginning of the next share some tokens. This prevents losing important information when a concept is split across two chunks.

**Q9.** What is the difference between dense retrieval and sparse retrieval?
**Answer:** Dense retrieval (embeddings) is meaning-based — it finds semantically similar text even with different words. Sparse retrieval (BM25/TF-IDF) is word-overlap-based — it finds exact keyword matches. Dense misses exact codes; sparse misses paraphrases. The course recommends combining both (hybrid) in production.

**Q10.** What is a vector store (vector database)? Why is Chroma recommended for AIMS labs?
**Answer:** A vector store stores (vector, text, metadata) tuples and answers nearest-neighbor queries efficiently. Chroma is recommended because it is open source, pure Python, runs embedded (no server), works on Colab and laptops, and is sufficient for AIMS-scale corpora.

**Q11.** Explain the two-stage retrieval (reranking) approach.
**Answer:** Stage 1: A bi-encoder retrieves top-50 candidates quickly using cheap embedding similarity. Stage 2: A cross-encoder reranks those 50 into top-5 by reading the query and chunk together — much slower but much more accurate. This typically lifts answer quality by 5–15 points.

**Q12.** What should the LLM do when the retrieved context doesn't contain the answer?
**Answer:** The LLM must be explicitly instructed to refuse and say "I don't know from the provided documents." Without this instruction, it will hallucinate a plausible-sounding answer even when the context is silent on the topic.

**Q13.** What happens if you use an English-only embedding model on French/Wolof text?
**Answer:** Performance degrades silently — the embeddings become less meaningful, retrieval becomes less accurate, and the system appears to work but returns wrong or irrelevant chunks. You won't get an error; you just get bad results.

**Q14.** What causes a "stale index" in RAG, and how do you fix it?
**Answer:** A stale index happens when source documents are updated but the embeddings are not re-generated. The system then answers from an old version of the document. The fix is to re-embed whenever documents change.

**Q15.** Compare a standalone LLM with a RAG-powered LLM when asked about the AIMS academic regulations.
**Answer:** The standalone LLM will either hallucinate a plausible answer (inventing rules) or refuse, because AIMS 2025–2026 regulations were never in its training data. The RAG-powered LLM retrieves the relevant passages from the official PDF and generates an answer grounded in those passages, admitting uncertainty when the document is silent.

**Q16.** What is the "recursive" chunking strategy and when should you use it?
**Answer:** Recursive chunking splits text first on double line breaks (paragraphs), then single line breaks, then sentences, then words — progressively finer as needed to hit the target size. It is best for narrative documents like the AIMS handbook because it respects natural text structure.

**Q17.** In the Senegalese health example, the query says "Dakar" but the correct document mentions "Pikine." How does the embedding model still find the right document?
**Answer:** The embedding model works on semantic meaning, not exact words. "Vaccinate a child in the Dakar area" and "centres de santé de Pikine offering free vaccination for children" have close semantic vectors — they are about the same concept — so cosine similarity ranks them close together, finding the right document.

---

## I-3 — cours_aims_module2_session7
### LoRA & QLoRA Fine-Tuning

---

### I-3-a — Summary and Important Points to Remember

#### Course Objective

By the end of Session 7, you will understand when to fine-tune (versus prompt or retrieve), how LoRA and QLoRA make it affordable, how to prepare data, and how to run a fine-tune on a free Google Colab GPU.

The guiding question: *"How would you adapt a 1B model to answer student questions in a fixed AIMS-Senegal style, using only a free Colab GPU?"*

---

#### Why Fine-Tuning Exists

**RAG gives the model new knowledge.** But the model's behavior, style, and output format are still those of the base model. Sometimes that is not enough.

Fine-tuning **changes how the model behaves** — its tone, its output format, its domain expertise. It does NOT add new factual knowledge (use RAG for that).

**Cases where fine-tuning helps:**
- AIMS academic tone in French, every time
- Strict JSON output for downstream parsing
- Wolof reformulation of medical advice
- A triage agent that follows a specific health protocol
- A code agent that uses internal AIMS-IT conventions

**Cases where fine-tuning is the WRONG tool:**
- The model needs to know new facts → use RAG
- One-off tasks → just write a better prompt
- Rapidly changing data (regulations) → RAG
- Missing data quality → fix data first

---

#### Prompting vs RAG vs Fine-Tuning

**Decision Tree:**
1. Is prompting enough? → Use prompting only
2. Need new knowledge? → Use RAG
3. Need new behavior/style? → Use fine-tuning

**Heuristic:** Fine-tune only when prompting and RAG cannot reach the bar. It costs more and ages faster than a prompt change.

---

#### Why Full Fine-Tuning Is Impractical

When you fine-tune a model, you traditionally update **all** the weights. For a 7B parameter model with Adam optimizer, this requires approximately:

**VRAM ≈ 16 × number_of_parameters bytes = 16 × 7B = 112 GB**

This is completely out of reach for a Colab T4 (16 GB) or even an A100 (40–80 GB) for large models.

**The insight:** We want to change only a small fraction of the model's behavior, not 100% of it. Updating all parameters to do this is extremely wasteful.

---

#### LoRA: The Core Idea

**LoRA (Low-Rank Adaptation)** is an elegant solution:

1. **Freeze** all the original model weights (W) — don't touch them
2. Add a small **low-rank update**: ΔW = B × A
   - A is a small matrix of shape (rank r × d)
   - B is a small matrix of shape (d × rank r)
   - r is much smaller than d (e.g. r = 8, d = 1024)
3. The new effective weight is: W' = W + (α × B × A)

**Why does this work?** Empirically, the useful update to a model's behavior is approximately low-rank — you don't need the full d × d degrees of freedom.

**Parameter count intuition:**
- Full update on one layer: d² parameters
- LoRA update with rank r: only 2rd parameters
- With d = 1024 and r = 8: ratio = 16/1024 ≈ 1.56%
- Combined across the model: you train only ~0.1 to 1% of parameters

**Practical LoRA hyperparameters:**
- Rank r ∈ {8, 16, 32} (higher = more capacity)
- Alpha α ∈ {16, 32, 64} (often α = 2r)
- Dropout ∈ {0.05, 0.1}
- Apply to: attention projections (q_proj, k_proj, v_proj, o_proj) and often MLP layers

---

#### QLoRA: LoRA on a Quantized Base

**QLoRA** takes LoRA one step further:

1. Load the base model in **4-bit (NF4) quantization** — this compresses the model dramatically
2. Freeze it
3. Attach LoRA adapters (in FP16 full precision) to the relevant layers
4. Train **only the adapters**

**The breakthrough result:**
- A 7B model that requires ~24 GB with LoRA now fits in ~8 GB with QLoRA
- A 70B model that was out of reach for a single GPU now fits in ~24 GB

This means you can fine-tune a 7B model on a free Colab T4 (16 GB GPU).

**LoRA vs QLoRA summary:**

| | LoRA | QLoRA |
|---|---|---|
| Base model precision | FP16/BF16 | 4-bit (NF4) |
| Memory for 7B | ~24 GB | ~8 GB |
| Memory for 70B | Out of reach | ~24 GB |
| Quality vs full fine-tuning | Nearly equivalent | Nearly equivalent |
| Best for | Laptops with mid GPUs, A100 | Free Colab T4, single 24 GB GPU |

---

#### Parameter-Efficient Training

The key idea: instead of training 100% of the model, train only the small LoRA adapters (~0.5–1% of parameters). Everything else is frozen. This is called **Parameter-Efficient Fine-Tuning (PEFT)**.

The saved adapters are a small file that you can load on top of any copy of the base model. You don't need to share the whole model — just the adapters.

---

#### Why Unsloth?

**Unsloth** is a library that makes QLoRA even faster:
- 2 to 5× faster LoRA/QLoRA training
- 80% less memory during training
- Custom CUDA kernels for attention and MLP
- Full Hugging Face compatibility (works with Trainer, PEFT, TRL)

**Practical result:** Fine-tune Qwen3-0.6B on a free Colab T4 in a short lab session with margin to spare.

**Caveat:** Unsloth optimizes a curated list of architectures — always check the support table for your exact model.

---

#### Data Preparation

**The instruction-response format:** Each training example is a JSON object with three fields:
- `instruction` — what to do
- `input` — optional input text
- `output` — the expected response

*Example 1 (Senegalese health):*
```json
{
  "instruction": "Reformule en wolof simple ce conseil médical.",
  "input": "Buvez beaucoup d'eau pendant les fortes chaleurs.",
  "output": "Naan ndox bu bari ci jamono ji tang yi."
}
```

*Example 2 (AIMS academic):*
```json
{
  "instruction": "Résume cet extrait pour un étudiant Master 2 d'AIMS.",
  "input": "Self-attention computes a weighted sum of values...",
  "output": "Self-attention pondère les valeurs par la similarité query-clé."
}
```

---

#### Quality Over Quantity

**The LIMA finding:** 1,000 high-quality examples typically beat 100,000 noisy ones for instruction-style fine-tuning.

**Quality checklist:**
- Answers are correct and consistent
- Style matches your target
- Instructions are diverse
- No leakage from the evaluation set

---

#### Synthetic Data

When datasets don't exist (especially for African languages like Wolof, Pulaar, Bambara, Hausa, Swahili), generate them synthetically:

**1. Self-Instruct (Wang et al., 2022):** Start with ~175 hand-written seed instructions. Ask a strong LLM to generate new instructions and responses. Filter, deduplicate, repeat.

**2. Distillation from a strong teacher:** Use a larger model (Qwen2.5-72B, Llama 3 70B, GPT-4, Claude) as teacher to generate instruction-response pairs, then fine-tune a smaller student (0.6B–7B) on them.
- *AIMS example:* Teacher = Qwen2.5-72B prompted to act as an AIMS professor. Student = Qwen3-0.6B fine-tuned on 5,000 teacher QA pairs.

**3. Evol-Instruct (Xu et al., 2023):** Take a base instruction and progressively make it more complex — add constraints, deepen reasoning, broaden context.
- Evolution example:
  1. "Explique le palu."
  2. "Explique le palu pour un agent de santé en Casamance."
  3. "Explique le palu pour un agent de santé en Casamance, avec 3 conseils pratiques en wolof."

**Quality filtering:**
- LLM-as-a-Judge: score (instruction, response) pairs on relevance and quality
- Deduplication by hash and semantic similarity
- IFD score: keep middle-range examples (not too easy, not too hard)

**License caution:** Check the teacher model's license before using synthetic data. OpenAI and others restrict competing-model training. Open-weight models (Qwen, Mistral, Llama) are usually more permissive.

---

#### Common Pitfalls

| Pitfall | Description | Fix |
|---|---|---|
| **Catastrophic forgetting** | Aggressive fine-tuning wipes useful general behavior | Use smaller learning rates and shorter training |
| **Eval set leakage** | Evaluation prompts leaked into training data | Hold out eval prompts before generating synthetic data |
| **Tokenizer drift** | Different chat templates at training vs inference | Use the same template in both places |
| **Tokenizer mismatch** | African scripts (Wolof, Pulaar) tokenize poorly with English-centric tokenizers | Check token counts and BPE merges before training |

---

#### The Unsloth Workflow (Lab Steps)

1. Install Unsloth, load base model in 4-bit (load_in_4bit=True)
2. Attach LoRA adapters with `FastLanguageModel.get_peft_model()`
3. Prepare instruction-response dataset, convert to chat template format
4. Train with TRL SFTTrainer for 1–2 epochs
5. Save adapters, run inference, compare before/after

---

#### Important Examples from the Course

**AIMS Lab scenario:** Fine-tune Qwen3-0.6B on a free Colab T4 with 200–500 instruction-response pairs on topics like:
- AIMS academic FAQ in French
- Simple medical advice reformulated in Wolof
- Agriculture tips for Senegalese farmers
- Code explanations for Master 2 students

What to look for after fine-tuning: tone shift, fewer unnecessary refusals, more consistent format. **Factual depth stays similar** because fine-tuning doesn't add new knowledge.

---

#### Final Takeaways

1. Fine-tune to **change behavior**, not to add knowledge.
2. LoRA freezes W and learns a low-rank update BA, training ~1% of parameters.
3. QLoRA = LoRA on a 4-bit base. It makes large open models trainable on a single accessible GPU.
4. Unsloth makes a small-model QLoRA fine-tune fit on a free Colab T4.
5. Quality of the dataset dominates. 1,000 clean examples beat 100,000 noisy ones.
6. For African languages: build seed sets, then expand with Self-Instruct, distillation, or Evol-Instruct, with quality filtering.

---

### I-3-b — Quiz Questions and Answers

**Q1.** What is the main purpose of fine-tuning? What does it change that prompting cannot?
**Answer:** Fine-tuning changes the model's behavior, style, tone, and output format permanently. Prompting influences only a single call. Fine-tuning embeds behavioral changes into the model weights, so the behavior persists across all future calls without needing a complex prompt.

**Q2.** According to the decision tree, when should you choose fine-tuning over RAG?
**Answer:** Choose fine-tuning when you need to change the model's **behavior** (style, tone, output format, domain conventions). Choose RAG when you need the model to access **new factual knowledge** from documents. Fine-tuning is the wrong tool for knowledge; RAG is the wrong tool for behavior.

**Q3.** Why is full fine-tuning of a 7B model impractical on a standard GPU?
**Answer:** Full fine-tuning requires storing weights, gradients, and optimizer states for all parameters. For a 7B model with Adam optimizer, this requires approximately 112 GB of VRAM — far exceeding a Colab T4 (16 GB) or even an A100 (40–80 GB).

**Q4.** Explain LoRA in simple terms. What does it freeze and what does it train?
**Answer:** LoRA freezes all original model weights and adds two small matrices (A and B) alongside the frozen weights. Only these small matrices are trained. The final effective weight is W + α×BA. Because the matrices are low-rank (small), only ~0.1–1% of total parameters are trained.

**Q5.** What is the rank (r) in LoRA and what happens when you increase it?
**Answer:** Rank r controls the size of the low-rank matrices (A has r×d elements, B has d×r elements). Increasing r adds more trainable parameters and more capacity to learn complex behavior — but at higher memory and computation cost. Typical values: r ∈ {8, 16, 32}.

**Q6.** How does QLoRA differ from LoRA?
**Answer:** QLoRA loads the base model in 4-bit (NF4) quantization instead of full precision FP16. This dramatically reduces memory usage (e.g., 7B model: ~24 GB with LoRA → ~8 GB with QLoRA), making large models trainable on small GPUs like a free Colab T4.

**Q7.** What is NF4 quantization in QLoRA?
**Answer:** NF4 (NormalFloat 4-bit) is a quantization format that compresses each model weight from 16 bits to 4 bits. This reduces the model's memory footprint by approximately 4×, making it possible to load large models on small GPUs.

**Q8.** What does Unsloth add to the QLoRA workflow?
**Answer:** Unsloth provides 2–5× faster training and 80% less memory usage through custom CUDA kernels. It is fully compatible with Hugging Face's Trainer, PEFT, and TRL. Practically, it allows fine-tuning Qwen3-0.6B on a free Colab T4 within a short lab session.

**Q9.** What are the three fields in the instruction-response format for training data?
**Answer:** (1) `instruction` — what the model should do; (2) `input` (optional) — the input text to process; (3) `output` — the expected response. This is the standard schema for supervised fine-tuning.

**Q10.** What did the LIMA paper show about data quantity vs. quality?
**Answer:** 1,000 high-quality examples typically beat 100,000 noisy ones for instruction-style fine-tuning. Data quality dominates quantity.

**Q11.** Explain the Self-Instruct technique for synthetic data generation.
**Answer:** Self-Instruct starts with ~175 seed instructions written by hand. A strong LLM is then prompted to generate new instructions and responses similar to the seeds. The generated data is filtered and deduplicated, then the process repeats. This expands a small seed set into thousands of training examples.

**Q12.** What is distillation from a strong teacher, and give an AIMS-specific example.
**Answer:** A large, capable model (the teacher, e.g., Qwen2.5-72B) generates instruction-response pairs in the desired style. A smaller student model (e.g., Qwen3-0.6B) is then fine-tuned on these pairs. AIMS example: Teacher is Qwen2.5-72B prompted to act as an AIMS professor; student is Qwen3-0.6B fine-tuned on 5,000 of the teacher's QA pairs to produce a deployable AIMS assistant.

**Q13.** What is catastrophic forgetting and how do you mitigate it?
**Answer:** Catastrophic forgetting happens when aggressive fine-tuning overwrites the model's useful general abilities (e.g., it forgets how to write good English after being trained only on French). Mitigation: use smaller learning rates and shorter training runs.

**Q14.** You fine-tune a model on new behavioral data. A week later, you deploy it but the answers are inconsistent. What might be the tokenizer-related problem?
**Answer:** Tokenizer drift — the chat template used at training time differs from the one used at inference time. Even a subtle difference in how messages are formatted can collapse output quality. The fix is to use the exact same chat template in both training and inference.

**Q15.** After fine-tuning a model on Senegalese AIMS data, a student notices the model no longer answers well in English. What happened and what should have been done differently?
**Answer:** Catastrophic forgetting occurred — the fine-tune overwrote general capabilities. The fix is to use a smaller learning rate, fewer epochs, or to include diverse examples that maintain general capability alongside the AIMS-specific examples.

**Q16.** What is the difference between attending to q_proj/k_proj/v_proj vs training all layers in LoRA?
**Answer:** Applying LoRA only to attention projections trains fewer parameters (attention-only). Applying LoRA to both attention and MLP layers (gate_proj, up_proj, down_proj) typically achieves better results. The course notes that attention+MLP usually beats attention-only.

**Q17.** Why should you check a teacher model's license before using it to generate synthetic training data?
**Answer:** Some providers (like OpenAI) restrict using their model outputs to train competing models. Using synthetic data from a model with such restrictions to train your own model could violate their terms of service. Open-weight models (Qwen, Mistral, Llama) are generally more permissive but you should always verify the specific license.

---

## I-4 — cours_aims_module2_session8
### Evaluation of LLM Pipelines

---

### I-4-a — Summary and Important Points to Remember

#### Course Objective

By the end of Session 8, you will know how to **rigorously evaluate** the outputs of an LLM pipeline using classic metrics, LLM-as-a-Judge, rubrics, and RAGAS.

The guiding question: *"For the AIMS-Senegal assistant you will build, what is the smallest set of metrics that tells you it is good enough to ship to a real student?"*

The core message of the session: **"Measure before you trust. Without evaluation, 'it works on my prompt' is not engineering."**

---

#### Why Evaluation Is Important

You have now learned to prompt, retrieve, and fine-tune. But **does any of this actually work?** You cannot ship a system to real users — like AIMS students — without evidence that it is reliable.

A system that seems to work on a few demo questions may fail badly on real queries. Evaluation forces you to discover failures before users do.

---

#### Why Evaluating LLMs Is Hard

**Classical ML evaluation is simple:** For a classifier, accuracy is a number. A label is either right or wrong.

**LLM outputs are different:**
- Answers are open-ended text
- Many correct answers exist
- Style and tone matter
- Factual correctness depends on context

*AIMS example:* The question "Quel est le but du module GAAI?" has at least three valid answers:
- "Apprendre à déployer des LLM en Afrique."
- "Former des ingénieurs IA appliquée."
- "Maîtriser prompting, RAG et fine-tuning."
All three are valid. **Exact-match scores 0 for all of them.**

---

#### The Three Dimensions of LLM Quality

Every evaluation framework should measure at least:

| Dimension | What it means |
|---|---|
| **Correctness** | Are the facts right? |
| **Groundedness** | Does the answer use the retrieved context? |
| **Style** | Is the format and tone appropriate? |

*Concrete example:* A Wolof health assistant might answer in correct French (style fail), or in fluent Wolof but with the wrong dosage (correctness fail), or quote an unrelated section of the protocol (groundedness fail).

---

#### Exact Match (EM)

- **Definition:** 1 if prediction equals reference exactly, 0 otherwise.
- **When it works:** Tasks with a unique short answer: dates, numbers, named entities, structured extraction.
- **When it fails:** Open answers. "Dakar" vs "la ville de Dakar" scores 0.

---

#### Token-Level F1

- **Definition:** Treat the answer as a bag of tokens. Compute precision, recall, and F1 against the reference.
- **When it works:** SQuAD-style short QA — more tolerant than EM.
- **Limit:** Still surface-form. "poisson" vs "thiof" (same fish, different names) scores 0.

---

#### BLEU

- **Definition:** N-gram precision of prediction against one or more references. Originally designed for machine translation.
- **When to use:** Translation tasks (e.g. English → Wolof) where token-level overlap is meaningful.
- **Shared limit with ROUGE:** Both reward surface overlap, not meaning. A correct paraphrase scores low.

---

#### ROUGE

- **Definition:** N-gram and longest-common-subsequence **recall**. Standard for summarization.
- **When to use:** Summarizing AIMS lecture notes, ANSD reports, scientific abstracts.
- **Same limit:** Rewards surface overlap. Do not use as the only metric.

---

#### Perplexity

- **Definition:** Measures how "surprised" the model is by held-out text. Lower is better.
- **Formula concept:** exp(−average log probability of each token)
- **Use for:** Comparing two language models on the same corpus; quick sanity check after fine-tuning.
- **Do NOT use for:** Ranking models on downstream tasks, evaluating instruction-following, comparing models with different tokenizers (numbers are not comparable across different tokenizations).
- **Heuristic:** "Perplexity is a thermometer. Useful, but does not tell you the meal is good."

---

#### Summary of Classic Metrics

| Metric | Best for | Main weakness |
|---|---|---|
| Exact Match | Short factual answers | Any rephrasing scores 0 |
| Token F1 | SQuAD-style short QA | Ignores meaning, only surface |
| BLEU | Translation | Rewards copy, punishes paraphrase |
| ROUGE | Summarization | Surface n-gram overlap |
| Perplexity | LM vs LM on same corpus | Not comparable across tokenizers |

**Take-away:** Use these metrics to **detect regressions**, not to claim absolute quality.

---

#### LLM-as-a-Judge

**Principle:** Use a strong LLM (GPT-4, Claude, Llama 3 70B) to score another LLM's output against a reference, a rubric, or a context.

**Why it's used:**
- Scales to thousands of examples
- Handles paraphrase and meaning (unlike EM/BLEU/ROUGE)
- Lets you score open-ended answers
- Cheaper than human labeling

**How it works:** You give the judge LLM: the question, the model's answer, the reference answer, and the retrieved context. The judge scores on multiple dimensions and explains its reasoning.

---

#### Known Biases of LLM-as-a-Judge

| Bias | Description |
|---|---|
| **Position bias** | Judges tend to prefer the first answer in pairwise comparison |
| **Verbosity bias** | Judges over-reward longer answers even when the short one is correct |
| **Self-preference bias** | A model judging outputs from the same family rates them higher |

**Mitigations:**
- Shuffle order in pairwise comparisons
- Use a rubric (not a free score)
- Cap output length on both judged model and judge
- Use majority vote of 3 judges
- Calibrate with 20 human-labeled samples
- Use a **different** judge model than the one being scored

---

#### Rubrics

A **rubric** is a structured scoring guide that tells the judge exactly what to score and how.

**Example rubric for the AIMS RAG (from the course):** Score 1–5 on four axes:
1. **Correctness:** Is the answer factually right given the AIMS document?
2. **Groundedness:** Are claims supported by the retrieved chunks?
3. **Refusal calibration:** Does the model say "I don't know" when it should?
4. **Style:** Is the language register appropriate (academic French)?

**Ship-ready threshold:** A pipeline that scores ≥ 4.0 on Correctness AND ≥ 4.0 on Groundedness is considered ready for an internal AIMS pilot.

---

#### RAGAS

**RAGAS** is an open-source framework that automates LLM-as-a-Judge for RAG pipelines. It standardizes evaluation with named metrics:

| RAGAS Metric | What it measures |
|---|---|
| **Faithfulness** | Are the model's claims supported by the retrieved context? |
| **Answer relevancy** | Does the answer actually match the question? |
| **Context precision** | Were the retrieved chunks actually used in the answer? |
| **Context recall** | Were the necessary chunks actually retrieved? |

**Why RAGAS matters:** It standardizes vocabulary across teams, makes evaluations reproducible, and integrates well with LangChain and LlamaIndex.

---

#### Failure Analysis

After running evaluation, for every failure case you should write one sentence: **which lever fixes it?**

| Failure pattern | What it signals | Fix |
|---|---|---|
| High correctness, low groundedness | Model knew the fact but ignored your context | Prompt change — stronger instruction to use context |
| Low refusal score | Hallucinates outside the corpus | Prompt change — add explicit refusal rule |
| Low style score | Tone is wrong (e.g., wrong language register) | Fine-tuning fix |
| Low context recall | Wrong chunks retrieved | Retrieval change — adjust chunking or embedder |

---

#### Final Takeaways

1. LLM eval is harder than classical ML eval: many right answers, style and grounding matter.
2. Classic metrics (EM, F1, BLEU, ROUGE, perplexity) detect regressions but do not measure quality.
3. LLM-as-a-Judge scales human evaluation, but requires rubrics, randomization, and a different judge model.
4. Always evaluate three dimensions: correctness, groundedness, style.
5. Evaluation must include refusal behavior and failure-case analysis.
6. RAGAS is a good standardization layer for RAG pipelines.

---

### I-4-b — Quiz Questions and Answers

**Q1.** Why is evaluating LLMs harder than evaluating classical ML classifiers?
**Answer:** Classical ML classifiers have a single correct label — accuracy is a number. LLM outputs are open-ended text with many valid answers. Style, tone, and groundedness also matter. A single metric cannot capture all dimensions of quality.

**Q2.** What are the three dimensions of LLM quality that every evaluation framework should measure?
**Answer:** (1) **Correctness** — are the facts right? (2) **Groundedness** — does the answer use the retrieved context? (3) **Style** — is the format and language register appropriate?

**Q3.** What is Exact Match (EM) and what is its main weakness?
**Answer:** Exact Match gives a score of 1 if the prediction equals the reference exactly, and 0 otherwise. Its main weakness is that any rephrasing — even if semantically correct — scores 0. "Dakar" vs "la ville de Dakar" scores 0, even though they refer to the same place.

**Q4.** What is Token-Level F1 and when is it better than Exact Match?
**Answer:** Token F1 treats the answer as a bag of tokens and computes precision, recall, and F1 against the reference. It is more tolerant than EM because partial matches get credit. It is useful for SQuAD-style short QA. However, it still misses semantic equivalences — "poisson" vs "thiof" scores 0 even though they mean the same thing.

**Q5.** What is BLEU designed for, and what is its shared limitation with ROUGE?
**Answer:** BLEU measures n-gram precision and was originally designed for machine translation. ROUGE measures n-gram recall for summarization. Both reward surface-level word overlap and penalize correct paraphrases — a semantically perfect answer in different words will score low.

**Q6.** What does perplexity measure and when should you NOT use it?
**Answer:** Perplexity measures how "surprised" the model is by held-out text (lower = better). Do NOT use it to rank models on downstream tasks, evaluate instruction-following, or compare models with different tokenizers (numbers are not comparable). Use it only as a quick sanity check after fine-tuning, comparing two models on the same corpus.

**Q7.** Explain LLM-as-a-Judge in simple terms.
**Answer:** LLM-as-a-Judge uses a strong LLM (like GPT-4 or Llama 3 70B) to evaluate another LLM's output. You give the judge the question, the answer, the reference, and the context. The judge scores the answer on multiple dimensions and explains its reasoning. This scales better than human evaluation and handles paraphrase, unlike classic metrics.

**Q8.** What is "position bias" in LLM-as-a-Judge and how do you mitigate it?
**Answer:** Position bias means the judge tends to prefer the first answer shown in a pairwise comparison regardless of quality. Mitigation: always randomize the order of answers when comparing two models.

**Q9.** What is "verbosity bias" in LLM-as-a-Judge?
**Answer:** Verbosity bias means judges over-reward longer, more elaborate answers — even when a short, correct answer is better. Mitigation: use a rubric with explicit criteria rather than a free score, and cap output length on both the judged model and the judge.

**Q10.** Why should you use a different judge model from the one being evaluated?
**Answer:** To avoid "self-preference bias" — a model tends to rate outputs from its own family or architecture higher than outputs from other models. Using a different judge avoids this systematic distortion.

**Q11.** What is a rubric in the context of LLM evaluation?
**Answer:** A rubric is a structured scoring guide that defines exactly what dimensions to score (e.g., correctness, groundedness, refusal calibration, style) and how to assign scores (e.g., 1–5 scale with criteria for each level). Using a rubric makes the judge's scoring more consistent and reduces free-form interpretation.

**Q12.** What are the four RAGAS metrics for RAG evaluation?
**Answer:** (1) **Faithfulness** — are the model's claims supported by the retrieved context? (2) **Answer relevancy** — does the answer match the question? (3) **Context precision** — were the retrieved chunks actually used? (4) **Context recall** — were the necessary chunks actually retrieved?

**Q13.** A RAG pipeline scores high on correctness but low on groundedness. What does this mean and how do you fix it?
**Answer:** It means the model knew the correct answer from its training data but ignored the retrieved context — it answered from memory rather than from the document. Fix: strengthen the prompt instruction to explicitly use only the provided context, and add a rule to refuse when the context doesn't support the answer.

**Q14.** A RAG pipeline scores low on "refusal calibration." What is happening?
**Answer:** The model is hallucinating answers when it should be saying "I don't know." It fails to recognize when the retrieved context doesn't contain the answer and invents something plausible instead. Fix: add an explicit refusal instruction in the prompt: "If the context does not contain the answer, say so."

**Q15.** What are the four metrics in the AIMS RAG rubric from the course?
**Answer:** (1) Correctness — is the answer factually right given the AIMS document? (2) Groundedness — are claims supported by retrieved chunks? (3) Refusal calibration — does the model say "I don't know" when appropriate? (4) Style — is the language register appropriate (academic French)?

**Q16.** According to the course, what score threshold on the AIMS rubric indicates a pipeline is "ship-ready"?
**Answer:** A pipeline that scores ≥ 4.0 on Correctness AND ≥ 4.0 on Groundedness (on a 1–5 scale) is considered ready for an internal AIMS pilot.

**Q17.** Why are classic metrics described as useful for "detecting regressions" but not for "claiming absolute quality"?
**Answer:** Classic metrics measure surface-level text similarity. They can tell you if a model got worse (regression) compared to a previous version — but a low EM or BLEU score doesn't mean the answer is wrong, and a high score doesn't mean the answer is correct. They cannot measure meaning, nuance, or whether the answer actually helps the user.

---

## I-5 — cours_aims_module3_session9
### Introduction to LLM Agents

---

### I-5-a — Summary and Important Points to Remember

#### Course Objective

By the end of Session 9, you will know what an LLM agent is, what makes it fundamentally different from a chatbot, and you will have built and run a first agent with a calculator and a Wikipedia search tool.

The guiding question: *"A market vendor in Sandaga asks: 'Quelle est la valeur du dollar en CFA aujourd'hui, et combien fait 250 USD?' A standalone LLM cannot answer. Your agent will."*

The core message: **"A chatbot answers. An agent acts."**

---

#### What Is an AI Agent?

**The simplest definition from the course:**

> **Agent = LLM + tools + memory + a loop that decides.**

An agent is not a smarter chatbot. It is an LLM with:
- **Hands** — tools it can use (APIs, code execution, databases, web search)
- **Eyes** — ability to observe results from those tools
- **A notebook** — memory of what it has done

**The shift:** Instead of guessing and generating text, the agent acts on the world to find the actual answer.

---

#### Chatbot vs Agent

| | Chatbot | Agent |
|---|---|---|
| What it does | Produces text answers | Uses tools, holds memory, plans steps |
| Knowledge source | Training data only | Training data + real-time tools |
| Can it calculate? | Only in its head (unreliable) | Yes — calls a calculator tool |
| Can it look up today's data? | No (knowledge cutoff) | Yes — calls a live API |
| Can it plan multi-step tasks? | No | Yes |

**The Sandaga Story (key example):**
- Query: "Convert 250 USD to CFA at today's rate."
- **Standalone LLM:** Quotes a fixed exchange rate from training time. Wrong. Confident anyway.
- **Agent:** (1) Reads the question; (2) Calls a live exchange-rate API; (3) Multiplies 250 by the live rate; (4) Formats the answer in CFA.
- The model stopped guessing. It acted on the world to find out.

---

#### Three Things an Agent Does That a Chatbot Cannot

1. **Use tools** — Calls APIs, runs code, queries databases, browses the web.
2. **Hold memory** — Remembers past steps in the same session, and sometimes across sessions.
3. **Plan steps** — Breaks a complex task into ordered subtasks, decides what to do next from observations.

---

#### Agent Architecture — The Four Parts

| Part | Role |
|---|---|
| **LLM (Brain)** | Chooses an action or produces the final answer |
| **Controller** | Validates the JSON output, calls tools, stops infinite loops |
| **Tools** | Do work outside the model (external APIs, code execution, databases) |
| **Trace** | Records every step — lets you debug the system |

---

#### The ReAct Loop

**ReAct (Reason + Act)** is the standard control loop for agents (from Yao et al., ICLR 2023).

One iteration of the loop:
1. **Observe** — Read the current state (the question, or the last tool's result)
2. **Reflect** — The LLM thinks: "What do I need to do next?"
3. **Act** — Call a tool, OR return the final answer

The loop repeats until the task is done.

**Why it works:** Forcing the model to write its reasoning before acting reduces hallucinated tool calls and makes the trace debuggable.

**ReAct trace for the Sandaga question:**
| Step | Content |
|---|---|
| Observation | "Convertis 250 USD en CFA au taux du jour." |
| Thought | I need today's USD/CFA rate. The calculator is for the multiplication. |
| Action | `exchange_rate(base="USD", quote="XOF")` |
| Observation | 605.30 |
| Thought | Now I multiply 250 by 605.30. |
| Action | `calculator("250 * 605.30")` |
| Observation | 151325.0 |
| Final | "250 USD font 151 325 FCFA au taux du jour (1 USD = 605,30 FCFA)." |

**The model never invented a rate. It asked. Twice. Then formatted.**

---

#### Function Calling: The JSON Contract

Instead of free-form text, the model emits a **JSON object** that names a function and its arguments. The runtime calls it and returns the result to the model.

*Example:*
```json
{
  "tool": "exchange_rate",
  "arguments": {
    "base": "USD",
    "quote": "XOF"
  }
}
```

**Why JSON?** It is parseable, validatable, and language-agnostic. The model's "intent" becomes machine-executable.

**Critical insight: The docstring is the prompt.** The tool description (docstring) is what the LLM reads to understand what the tool does and when to use it. A good docstring is crucial for the agent to call the right tool.

---

#### Tools

Tools are functions or services the agent can call. They do work outside the model:
- Calculator: evaluates arithmetic
- Wikipedia search: looks up factual information
- Exchange rate API: gets live currency rates
- Code executor: runs Python code
- Database query: retrieves structured data

In LangChain, a tool is created with the `@tool` decorator. The function's docstring becomes the description the LLM reads.

---

#### Memory

Memory lets the agent remember past steps. Three types mentioned in the module:
- **In-session (trace memory):** The conversation history and tool results within one session
- **Cross-session (persistent memory):** Information stored and retrieved across sessions
- **External (tool-based):** Reading from databases or files

Session 10 covers memory and workflows in more depth.

---

#### Frameworks

| Framework | What it does | When to use |
|---|---|---|
| **LangChain** | Tools + ReAct loop ready | Starting point for most agents |
| **LangGraph** | Tools + graph of nodes and edges | Custom workflows, production |
| **CrewAI** | A crew of agents with roles | Multi-agent role-play tasks |
| **Google ADK** | Conversational agent + Gemini | Google-stack apps, fast demos |

**Course default:** LangChain for basics in Session 9; LangGraph for workflows in Session 10.

**What LangChain gives for free:**
- `@tool` decorator with auto-schema
- ReAct loop ready out of the box
- Integrations with most LLM providers
- Visible reasoning traces for debugging

---

#### When Do You Really Need an Agent?

Use an agent when the task requires:
- Accessing real-time data (live prices, weather, APIs)
- Multi-step reasoning where each step depends on a previous result
- Using external computation (calculators, code, databases)
- Planning a sequence of actions

**African use cases from the course:**
- **Education:** AIMS tutor that solves math step by step with SymPy, checks each step
- **Health:** Triage agent pulling vital signs, checking Senegalese protocols, escalating to a clinician
- **Public services:** DGID assistant filling a tax form by reading rules and computing amounts in CFA
- **Agriculture:** Advisor reading ANACIM weather + ISRA bulletins + farmer's crop, returning 3 decisions in Wolof

---

#### Reading the Trace

The trace is your debugger. With `verbose=True` in LangChain, you see every Thought, Action, and Observation.

**Healthy signs:**
- Agent calls Wikipedia for facts, calculator for arithmetic (right tool, right job)
- Thoughts are short and concrete
- Loop terminates in 3–5 steps
- Final answer cites the numbers used

**Signs of trouble:**
- Agent loops on the same tool repeatedly
- Invents Wikipedia results that look plausible
- Skips arithmetic and "estimates" instead
- Trace gets longer than 8 steps

---

#### Agent Examples from the Course

**Lab agent (two tools):**
- Tool 1: Calculator (evaluates arithmetic expressions)
- Tool 2: Wikipedia search (French)
- Test questions:
  - "Quelle est la population de Dakar et combien d'habitants par km2 si la superficie est de 83 km2?"
  - "Quand AIMS Senegal a été fondé, et combien d'années nous séparent de cette date?"
  - "Donne la capitale du Mali et sa distance approximative de Dakar."

---

#### Final Takeaways

1. An agent is an LLM with tools, memory, and a loop. Nothing more, nothing less.
2. ReAct = Observe, Reflect, Act. Repeat until the task is done.
3. Tool calling is a JSON contract. The docstring is the prompt.
4. LangChain is the right starting point. LangGraph and CrewAI come next.
5. The trace is your debugger. Read it before you change anything.
6. Failures are pedagogical. Catalog them, fix the docstrings.

---

### I-5-b — Quiz Questions and Answers

**Q1.** What is the simplest definition of an LLM agent?
**Answer:** Agent = LLM + tools + memory + a loop that decides. An agent is an LLM that can use external tools, hold memory of past steps, and loop through observations and actions until a task is complete.

**Q2.** What are the three things an agent can do that a chatbot cannot?
**Answer:** (1) **Use tools** — call APIs, run code, query databases, browse the web; (2) **Hold memory** — remember past steps in the session and across sessions; (3) **Plan steps** — break a complex task into ordered subtasks and decide what to do next based on observations.

**Q3.** Explain the Sandaga story. Why can a standalone LLM not answer the currency conversion question correctly?
**Answer:** A standalone LLM only knows information from its training data — including exchange rates that existed at training time. By the time of the query, the rate has changed. The LLM will quote a stale rate confidently. An agent calls a live exchange-rate API to get today's actual rate, then calls a calculator to compute the conversion.

**Q4.** What are the four parts of an agent and what does each do?
**Answer:** (1) **LLM (Brain)** — chooses an action or produces the final answer; (2) **Controller** — validates the JSON output, calls tools, stops infinite loops; (3) **Tools** — do work outside the model (APIs, code, databases); (4) **Trace** — records every step for debugging.

**Q5.** Explain the ReAct loop. What are the three steps in each iteration?
**Answer:** ReAct stands for Reason + Act. Each iteration: (1) **Observe** — read the current state (question or last tool result); (2) **Reflect** — the LLM reasons about what to do next; (3) **Act** — call a tool or return the final answer. The loop repeats until the task is complete.

**Q6.** Why does forcing the model to write its reasoning before acting reduce errors?
**Answer:** Writing reasoning before acting forces the model to plan and justify its tool choice. This reduces hallucinated tool calls (calling the wrong tool or inventing results) and makes the trace debuggable — you can see exactly why the model made each decision.

**Q7.** What is function calling? Why is JSON used instead of free-form text?
**Answer:** Function calling is when the LLM emits a structured JSON object naming a function and its arguments, instead of generating free-form text. JSON is used because it is parseable, validatable, and language-agnostic — the model's "intent" becomes directly machine-executable without needing a text parser.

**Q8.** What is "the docstring is the prompt" principle?
**Answer:** The description (docstring) of each tool is what the LLM reads to understand when and how to use that tool. A clear, specific docstring is crucial — it acts like a mini-prompt that guides the agent's tool selection. A bad docstring causes the agent to call the wrong tool or misuse it.

**Q9.** What is LangChain and what does it give you for building agents?
**Answer:** LangChain is a Python framework for building LLM applications and agents. It provides: the `@tool` decorator with auto-schema, the ReAct loop ready out of the box, integrations with most LLM providers, and visible reasoning traces for debugging.

**Q10.** What is LangGraph and when should you prefer it over LangChain?
**Answer:** LangGraph is a framework that represents agent workflows as a graph of nodes and edges. It is better than LangChain for complex, custom multi-step workflows in production, where you need explicit control over the flow between agent steps.

**Q11.** What does "verbose=True" do in a LangChain AgentExecutor and why is it useful?
**Answer:** With `verbose=True`, LangChain prints every Thought, Action, and Observation in the trace as the agent runs. This lets you see exactly what the agent is doing at each step — crucial for debugging, understanding failures, and improving tool docstrings.

**Q12.** What are the healthy signs that a ReAct trace is working correctly?
**Answer:** The agent calls Wikipedia for facts and the calculator for arithmetic (right tool for right job); thoughts are short and concrete; the loop terminates in 3–5 steps; the final answer cites the numbers it used.

**Q13.** What are the signs of trouble in a ReAct trace?
**Answer:** The agent loops on the same tool repeatedly; it invents plausible-looking Wikipedia results; it skips arithmetic and "estimates" instead; the trace grows longer than 8 steps.

**Q14.** A student builds a currency agent. The agent calls the exchange_rate tool, gets the result, but then writes "approximately 150,000 CFA" instead of computing the exact value. What went wrong?
**Answer:** The agent skipped arithmetic — it estimated instead of calling the calculator tool. This happens when the calculator's docstring is unclear, or when the model doesn't recognize that multiplication is needed. Fix: improve the calculator tool's docstring and make the ReAct prompt explicitly instruct the model to use tools for all computation.

**Q15.** What is the difference between LangChain and CrewAI? When would you choose CrewAI?
**Answer:** LangChain is for building single agents with tools and a ReAct loop. CrewAI is for building a "crew" of multiple agents with different roles (like a researcher, a writer, a reviewer) working together. Choose CrewAI when your task benefits from multi-agent collaboration with specialized roles.

**Q16.** Give two African use cases where an agent is more appropriate than a simple RAG chatbot.
**Answer:** (1) DGID tax assistant that reads the user's situation, looks up the correct tax rule, AND computes the amount due in CFA — this requires both retrieval and calculation, which only an agent can do. (2) Agriculture advisor that reads live ANACIM weather data + ISRA bulletins, then provides specific decisions — this requires real-time tool access beyond what a static RAG index can provide.

**Q17.** After a lab session, a student finds that their agent always loops more than 8 steps before answering. What is likely wrong and how should they investigate?
**Answer:** The agent may be confused about which tool to use, getting poor results from tools, or failing to recognize when it has enough information to answer. Investigation: read the full trace (Thought-Action-Observation at each step) to find where the loop degenerates. Common causes: ambiguous docstrings, tools returning unhelpful results, or the system prompt not clearly instructing the agent to conclude when sufficient information is available.

---

# II — FINAL GLOBAL REVISION SHEET

---

## Most Important Concepts from Sessions 5–9

### One-Sentence Definitions

| Concept | Definition |
|---|---|
| **Prompt** | A specification of the desired output given to an LLM, including role, task, context, format, and constraints. |
| **Zero-Shot Prompting** | Asking the LLM to perform a task with no examples — instruction alone. |
| **Few-Shot Prompting** | Providing 2–5 input/output examples before the real query so the model imitates the pattern. |
| **Chain-of-Thought (CoT)** | Asking the model to reason step-by-step before giving the final answer. |
| **Self-Consistency** | Running the same prompt k times and taking the majority vote to reduce variance. |
| **Context Window** | The maximum number of tokens (input + output) an LLM can process in one call. |
| **Lost-in-the-Middle** | The empirical finding that models retrieve information less reliably from the middle of long contexts. |
| **RAG (Retrieval-Augmented Generation)** | A pipeline that retrieves relevant document chunks and injects them into the LLM's prompt to ground its answers in specific documents. |
| **Embedding** | A numerical vector representation of text that encodes its meaning; similar texts have close vectors. |
| **Cosine Similarity** | A measure of how close two embedding vectors are, used to find semantically similar chunks. |
| **Chunking** | Splitting a long document into smaller passages (chunks) that can be retrieved individually. |
| **Vector Store** | A database that stores text-embedding pairs and answers nearest-neighbor (similarity) queries fast. |
| **Dense Retrieval** | Meaning-based search using embeddings; finds paraphrases and synonyms. |
| **Sparse Retrieval (BM25)** | Lexical overlap search; finds exact keyword and code matches. |
| **Reranking** | A two-stage retrieval approach: fast bi-encoder selects top-50, slow cross-encoder refines to top-5. |
| **Fine-Tuning** | Updating model weights on a specific dataset to permanently change its behavior, style, or output format. |
| **LoRA** | Freezes original weights and adds small trainable low-rank matrices (A and B) to learn behavior changes with ~1% of parameters. |
| **QLoRA** | LoRA applied to a 4-bit quantized base model, enabling fine-tuning of large models on small GPUs. |
| **NF4 Quantization** | 4-bit number format that compresses model weights ~4×, reducing memory while preserving quality. |
| **Unsloth** | A library providing 2–5× faster LoRA/QLoRA training with 80% less memory via custom CUDA kernels. |
| **Instruction-Response Format** | Training data schema with fields: instruction, input (optional), output. |
| **Synthetic Data** | Training data generated by an LLM (Self-Instruct, distillation, Evol-Instruct) when real data is scarce. |
| **Catastrophic Forgetting** | Aggressive fine-tuning that overwrites previously learned capabilities. |
| **Exact Match (EM)** | Evaluation metric: 1 if prediction equals reference exactly, 0 otherwise. |
| **BLEU** | N-gram precision metric designed for machine translation. |
| **ROUGE** | N-gram recall metric standard for summarization. |
| **Perplexity** | Measures how "surprised" a model is by text; useful for comparing models on the same corpus. |
| **LLM-as-a-Judge** | Using a strong LLM to score another LLM's output against a rubric or reference. |
| **Rubric** | A structured scoring guide defining what dimensions to evaluate and how to assign scores. |
| **RAGAS** | Open-source RAG evaluation framework with metrics: faithfulness, answer relevancy, context precision, context recall. |
| **Agent** | An LLM with tools, memory, and a decision loop (ReAct) that acts on the world instead of just generating text. |
| **ReAct Loop** | Observe → Reflect → Act cycle that drives agent behavior until the task is complete. |
| **Function Calling** | The LLM emits a structured JSON object naming a tool and its arguments for the runtime to execute. |
| **Tool (in agents)** | A function or service the agent can call to do work outside the model (APIs, calculators, web search). |
| **Trace** | The complete record of an agent's Thoughts, Actions, and Observations — used for debugging. |
| **LangChain** | Python framework providing ReAct loop, @tool decorator, and LLM integrations for building agents. |
| **LangGraph** | Extension of LangChain for building complex workflows as graphs of nodes and edges. |

---

## Comparison Table: Prompting vs RAG vs Fine-Tuning vs Agents

| Dimension | Prompting | RAG | Fine-Tuning | Agents |
|---|---|---|---|---|
| **What it does** | Guides the model's output for a single call | Connects the model to external documents | Permanently changes model behavior/style | Enables the model to act, use tools, and plan |
| **What problem it solves** | Improving output quality and format | Answering questions about your documents | Adapting style, tone, output format | Multi-step tasks requiring real-world actions |
| **Knowledge source** | Model's training data | External retrieved documents | Training data (learned into weights) | Training data + real-time tools |
| **When to use** | Always as a first try | When model needs your specific documents | When behavior must change permanently | When task requires tools and planning |
| **Cost** | Low (just tokens) | Medium (indexing + retrieval) | High (compute + data preparation) | Variable (tokens + tool calls) |
| **Speed to implement** | Minutes | Hours | Days | Hours |
| **Persistent change?** | No (per call only) | No (retrieval at query time) | Yes (weights updated) | No (behavior via tools and prompts) |
| **Can add new facts?** | No | Yes | No (behavior only) | Yes (via tools) |
| **Can change style?** | Partially | No | Yes | Partially |
| **Can handle stale info?** | No | Yes (update index) | No | Yes (live tools) |
| **Main limitation** | Hallucination, context limit | Retrieval quality, chunking | Data quality, cost, forgetting | Tool reliability, loop failures |
| **Key technique** | Zero/few-shot, CoT, format | Chunk → Embed → Retrieve → Generate | LoRA/QLoRA on instruction-response data | ReAct loop + function calling |
| **AIMS example** | Summarize an abstract in 3 sentences | Answer AIMS regulation questions from PDF | AIMS academic tone in French at all times | Currency conversion + Wikipedia lookup |

---

## Top 30 Facts to Remember Before an Exam

1. **A prompt is a specification**, not a question — it includes role, task, context, format, and constraints.
2. **Escalate prompting techniques** in order: zero-shot → few-shot → CoT → self-consistency.
3. **3 to 5 few-shot examples** are enough; more wastes context without improving results.
4. **Role prompting** changes tone and depth, NOT factual accuracy — it does not fix hallucinations.
5. **CoT increases token cost**; avoid for trivial questions and latency-critical apps.
6. **Self-consistency**: use k ∈ {3, 5, 7}; beyond 7, gains plateau but cost keeps multiplying.
7. **Lost-in-the-Middle**: put critical instructions first and critical evidence last in long prompts.
8. **Prompting cannot give the model new knowledge** — knowledge cutoff and context limits are hard ceilings.
9. **RAG = retrieve relevant chunks, then let LLM answer** — retrieval is grounded, not memorized.
10. **Indexing happens once; querying happens every call** — this is the two-phase RAG architecture.
11. **Embeddings encode meaning, not words** — similar meanings → close vectors → high cosine similarity.
12. **Chunk size 300–800 tokens, overlap 50–100 tokens** — start recursive at 500/80.
13. **Multilingual embedder required** for French/Wolof/African language AIMS projects.
14. **Always add explicit refusal instruction** to RAG prompts — "If context is silent, say so."
15. **Fine-tuning changes behavior; RAG adds knowledge** — never confuse the two.
16. **Full fine-tuning of 7B requires ~112 GB VRAM** — impractical for most GPUs.
17. **LoRA trains only ~0.1–1% of parameters** by learning low-rank updates B×A alongside frozen weights.
18. **QLoRA = LoRA + 4-bit quantization** → 7B fits in ~8 GB; 70B fits in ~24 GB.
19. **Unsloth: 2–5× faster training, 80% less memory** — use for Colab T4 labs.
20. **Quality over quantity**: 1,000 clean examples beat 100,000 noisy ones (LIMA finding).
21. **Catastrophic forgetting** = fine-tune overwrites general capabilities — use smaller LR and fewer epochs.
22. **Same chat template at training AND inference** — mismatch collapses quality (tokenizer drift).
23. **EM, BLEU, ROUGE detect regressions** — they do not measure actual answer quality.
24. **LLM-as-a-Judge has three biases**: position bias, verbosity bias, self-preference bias.
25. **Mitigate judge bias**: shuffle order, use rubric, use a different model as judge.
26. **RAGAS four metrics**: faithfulness, answer relevancy, context precision, context recall.
27. **Three quality dimensions to always measure**: correctness, groundedness, style.
28. **Agent = LLM + tools + memory + loop** — "a chatbot answers, an agent acts."
29. **ReAct loop**: Observe → Reflect → Act → repeat until done — forcing reasoning before acting reduces errors.
30. **The docstring IS the prompt** for a tool — a bad docstring causes wrong tool selection.

---

*End of Complete Study Guide — GAAI-AIMS Sessions 5 to 9*
*Based exclusively on course materials by Dr. Papa-Séga WADE, AIMS Senegal*
