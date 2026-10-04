# Text Generation from Argument Graphs with User-Generated Content

**Master's Thesis Project**

Argument search engines return snippets that often miss the main claim and its reasons. Online debates such as those on **Kialo** have no plain-text version to summarise; they exist only as **argument graphs**, with a main claim and user-written premises that support or attack it.

This project explores how to turn these graphs into **concise ~90-word summaries** that can serve as snippets in argument search engines. It compares **three strategies** for passing the graph structure to a language model and **three models** that generate the summary, evaluated through human judgement.

> **Research question:** How can the structural features of user-generated argument graphs be incorporated into a coherent textual summary for snippet generation?

## Approach

**Kialo argument graph** → **Input strategy** (Depth-First · Divide & Conquer · JSON) → **Model** (GPT-4 Turbo · LLaMA-2 70B · BART-Large-CNN) → **Prompt chaining** (90-word limit) → **Human pairwise evaluation**


### Strategies
| Strategy | How the graph is given to the model |
|----------|-------------------------------------|
| **Depth-First (DF)** | The graph is linearised by depth-first traversal from the main claim and summarised as plain text |
| **Divide & Conquer (DC)** | Each claim is summarised with its premises, joined by connectors ("because" for support, "however"/"but" for attack). Each summary is passed as a prefix to the next step, so the result builds up recursively. |
| **JSON** | The graph is converted to JSON (nodes + typed edges), a condensed form of the Argument Interchange Format, and given to the LLM with a "graph summarizer" prompt |

### Models
| Model | Summary type | Context limit |
|-------|--------------|--------------:|
| GPT-4 Turbo (`gpt-4-1106-preview`) | Abstractive | 128k tokens |
| LLaMA-2 70B Chat | Abstractive | 4,096 tokens |
| BART-Large-CNN | Extractive-style | 1,024 tokens |

BART cannot take JSON input, which gives **8 model–strategy combinations**.

### Key techniques
- **Prompt engineering:** each prompt specifies a role, input and output, and prompts were refined iteratively using the Goal–Prompt–Evaluation–Iteration (GPEI) method.
- **Prompt chaining:** summaries over 90 words are sent back to the model with stricter instructions.
- **Chunking:** inputs larger than a model's context limit are split, and each chunk is summarised together with the summary of the previous chunk. For JSON, chunks keep each node together with its edges.

## Evaluation

| | |
|---|---|
| **Data** | 50 Kialo argument graphs, randomly sampled from 1,230 (4,509 I-nodes, 9,018 edges) |
| **Method** | Human pairwise comparison: annotators pick the summary with the better overall quality |
| **Scale** | 8 summaries per graph → 28 pairs per graph → **1,400 pairs** |
| **Annotators** | 6 graduate-level evaluators, each assigned one of 6 batches |
| **Interface** | Custom Flask web app showing each pair with a link to the original debate |


### Key findings
- ✅ **Abstractive summaries (GPT-4, LLaMA-2) were strongly preferred** over BART's extractive-style summaries.
- ✅ **GPT-4 Turbo performed best** with every strategy. Its lead is largest with JSON input.
- ✅ **JSON input worked best** for both LLMs, and it held up regardless of graph size.
- ❌ **Divide & Conquer did not consistently beat Depth-First.** On large graphs GPT-4 preferred DF, since it could read the whole debate at once, while LLaMA-2 preferred DC, because its smaller context meant DF had to be split into many chunks.

## Repository Structure

```
NLP-GENAI/
├── llm_input/
│   ├── arg_graphs/          # 50 Kialo argument graphs
│   ├── dfs_linearized/      # Depth-first text input
│   └── json_input/          # JSON input
├── code/
│   ├── gpt4/  llama2/  bart-cnn/   # Summarisation notebooks per model & strategy
│   ├── bart-large/  t5/            # Early experiments
│   ├── shorten_summaries/          # Prompt chaining for the word limit
│   └── Evaluation/                 # Pair creation, batching, result analysis
└── llm_output/
    ├── before_shorten/      # Raw model summaries
    └── after_shorten_final/ # Final summaries used in the evaluation
```

**Tech:** Python · OpenAI API · LLaMA-2 · Hugging Face Transformers · arguebuf · NetworkX · Flask · pandas · Plotly
