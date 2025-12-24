# Summary

🔴 **Note: These notes are based on my understanding of the paper.**

![image.png](image.png)

<aside>
💡

This is a citation from the original paper, which I have summarized here. For additional details, refer to the original source.

Please note that all tables and figures are taken from the original paper.

> [Argument Mining with Fine-Tuned Large Language Models](https://aclanthology.org/2025.coling-main.442/) (Cabessa et al., COLING 2025)
• Jérémie Cabessa, Hugo Hernault, and Umer Mushtaq. 2025. [Argument Mining with Fine-Tuned Large Language Models](https://aclanthology.org/2025.coling-main.442/). In *Proceedings of the 31st International Conference on Computational Linguistics*, pages 6624–6635, Abu Dhabi, UAE. Association for Computational Linguistics.
> 
</aside>

👉 To better understand of basic of AM, I recommend to read Detecting Arguments in CJEU Decisions on Fiscal State Aid which I summarize it too.

### 🗣️Tasks:

This paper reformulates all tasks of Argument mining as **text generation problems** by generating structured text outputs (lists of labels) 👇

1. ACC: The goal is to classify a specific segment of text (argument component) into a defined role, such as a **Major Claim**, **Claim**, or **Premise**.
2. AIR: The goal is to determine if a relationship exists between two components. Determine they are **related** or **non-related**.
3. ARC: The goal is to classifies identified relations, such as whether one component **supports** or **attacks** another.
4. ARIC (ARI+ARC): Jointly identifying a relation and classifying its type (e.g., support or attack).
5. ACC–ARI–ARC: A full "end-to-end" model.

---

### 😊 Example:

![0f5590ef.jpg](0f5590ef.jpg)

🔵 Practical Example of how process implement:

🔴 Note: this is additional from paper

For example, the prompt gives the model an essay with 20 numbered components (`<AC0>` to `<AC19>`).

Input: An essay about technology and creativity.

Instructions: Classify each as "MajorClaim", "Claim" or "Premise"

Output: The model generates a long list starting with `"Claim"`, then `"MajorClaim"`, then several `"Premise"` labels, ending with a final `"MajorClaim"` to match the 20 components it was given.

Comparison: By comparing the two lists of Ground truth AC types and Generated AC types can calculate **Macro F1 scores**. → These scores tell us how often the AI correctly identified a "Premise" versus a "Claim" compared to the human experts.

---

### 👩‍🏭 Fine-tuning:

For fine-tuning, we use QLoRA strategy. To understand it better first pay attention to these definition 👇

1. What is Fine-tuning?

⇒ Fine-tuning refers to the process of further training a pre-trained LLM on a specific downstream task.

1. What is quantized LLMs? 

⇒ Is a **compressed version** of a LLM. Quantization ****stores model weights with fewer bits by rounding them, reducing memory requirement while trying to keep it as smart as possible.

**NOW**

What is QLoRA ? 

⇒ Is an abbreviation of Quantized Low-Rank Adaptation. It is an efficient way to train massive AI models on normal computers. QLoRA is combination of Quantization + LoRA 👇

**HOW it WORKS**

1. Frozen n-bit Quantized Version: 
    
    In a standard model, weights are stored with high detail. QLoRA **quantizes** these weights down to just **4 bits**. By freezing, its weights not updated during training so it require 75% less memory.
    
2. Rank Decomposition Matrices (Low-Rank Adapters)
    
    When we fine-tune a model, we change $W → W'$ . Now that the main model is frozen, the model needs a place to learn new information. QLoRA injects **Adapters** (also known as LoRA).
    
    $$
    ⁍
    $$
    
    Since the model already knows almost everything so fine-tuning only needs to adjust **a few directions** in weight space so ΔW can be **low-rank** (Rank is a number of independent rows of matrix). So factorizes to:
    
    $$
    \Delta W = AB
    $$
    

where,

$$
A \in R^{d*r} , B \in R^{r*d}
$$

which $r << d$ .

1.  How they work together
    
    During fine-tuning, the model performs forward passes using $W + AB$, computes a loss, backpropagates gradients, and updates only the low-rank adapter matrices $A$ and $B$ using gradient descent, effectively learning $\Delta W$ while keeping the base model frozen.
    

---

### 🧐 Model:

The model that are used are:

1. LLaMA-3 / LLaMA-3.1
2. Gemma-2
3. Mistral
4. Phi-3
5. Qwen-2

---

### 🗃️ Dataset:

The benchmark datasets that are used are:

1. PE (Persuasive Essays)
2. AbstRCT (Scientific Abstracts)
3. CDCP (Debate Corpus)

---

### 💡Results:

By fine-tuning multiple LLMs, both quantized and non-quantized, the authors achieve SOTA results on argument mining tasks.

![image.png](image%201.png)

![image.png](image%202.png)

![image.png](image%203.png)

Notable Insights:

🟩 Fine-tuned LLMs can significantly outperform previous models in argument mining subtasks. 

🟩 Larger models (e.g., 70B parameters) generally perform better than smaller ones.

🟩 Quantized models (4-bit) perform **very close to full-precision models**, showing efficient fine-tuning is practical.

🟩 Structural tags and essay-level context help some tasks, especially ARI.

🟩 This generative formulation (predicting lists of labels/relations) is effective across all subtasks.