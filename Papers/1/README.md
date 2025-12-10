# Summary

🔴 **Note: These notes are based on my understanding of the paper.**

<p align="center">
<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/36f535d9-da41-4317-846c-053f1842baac" />
</p>

<aside>
💡

This is a citation from the original paper, which I have summarized here. For additional details, refer to the original source.

Please note that all tables and a figure are taken from the original paper.

> [Detecting Arguments in CJEU Decisions on Fiscal State Aid](https://aclanthology.org/2022.argmining-1.14/) (Grundler et al., ArgMining 2022)
• Giulia Grundler, Piera Santin, Andrea Galassi, Federico Galli, Francesco Godano, Francesca Lagioia, Elena Palmieri, Federico Ruggeri, Giovanni Sartor, and Paolo Torroni. 2022. [Detecting Arguments in CJEU Decisions on Fiscal State Aid](https://aclanthology.org/2022.argmining-1.14/). In *Proceedings of the 9th Workshop on Argument Mining*, pages 143–157, Online and in Gyeongju, Republic of Korea. International Conference on Computational Linguistics.
> 
</aside>

---

👉 To better understand this paper, it is important to first explore its focus.

### ⚖️ What is Legal Argumentation ?

An argumentation is a set of statements intended to convince the judge or jury that desired conclusion is correct under the law.

<img src="image.png" width="300" height="200">

So, argumentation acts as bridge between its two components, Premises and Conclusion. 

- Premise (P): is a sentence that gives a reason, evidence, or rule to support claim.
- Conclusion (C): is the main claim that follows from the premises.

### 👩‍⚖️ What is Argument Mining (AM)?

By using ML and NLP, tries to address these problems 👇

1. Find argumentative sentence in document
2. Classify them (premise, conclusion)
3. Predict relations between them (support, attack, etc.)

This paper focuses on STEP 3, **argument structure prediction** in judicial decisions by the Court of Justice of the European Union on Fiscal State Aid. 

---

### The relation between these statements is :

1. Support (SUP): The premise supports the conclusion
2. rebuttal (ATT): The conclusion attacks the premise
3. Support from Failure (SFF): The premise supports the conclusion because of absence, not presence, of evidence.
4. Undercut (INH): The premise attacks the other premise (citation)
5. Rephrase (REPH): The premise is rephrase of other premise.

<img src="0749087e-796f-4394-b80a-ac4d14587bff.png" width="400" height="400">

### 📝Example:

 For better understanding, I’ll bring an example :

 A judge is deciding whether a company received illegal State aid or not?

> P1: The tax exemption given to Company X reduces the charges the company would normally pay.
> 
> 
> P2: According to EU case law, a measure that reduces charges which companies normally bear can constitute State aid.
> 
> C1: Therefore, the tax exemption may constitute State aid.
> 

⇒ **P1** SUP **C1**

⇒ **P2** SUP **C1**

> P3: The government argues that the exemption simply compensates for a structural disadvantage.
> 
> 
> P4: However, the company has not provided evidence of any structural disadvantage.
> 
> C2: Therefore, the argument based on structural disadvantage must be rejected.
> 

⇒ **C2** ATT **P3**

⇒ **P4** SFF **C2**

> P5: The government also cites the “SolarWind” judgment to justify the exemption.
> 
> 
> P6: But that case concerned renewable energy subsidies, not tax exemptions.
> 
> C3: Thus, the “SolarWind” judgment is not applicable here.
> 

⇒ **P6** INH **P5** → P6 questions the logic of P5

If we have also,

> P1b: Company X received a reduction of its normal fiscal burden.
> 

⇒ **P1** REPH **P1b →** they say the same thing

---

<aside>
💡

Note: This paper enrich the  **Demosthenes** dataset introduced in below paper with an additional layer of annotation to capture the inferential connections between propositions.

> Giulia Grundler, Piera Santin, Andrea Galassi, Federico Galli, Francesco Godano,
Francesca Lagioia, Elena Palmieri, Federico Ruggeri, Giovanni Sartor, and Paolo
Torroni. 2022. Detecting Arguments in CJEU Decisions on Fiscal State Aid. In
ArgMining@COLING. 143–157.
> 
</aside>

---

### 📊 Demosthenes Dataset:

![image.png](image%201.png)

🟩 Every argumentative statement is annotated as either a **Premise** or a **Conclusion**.

🟩 Each statement has a unique identifier (such as $A_n$, $B_n$).

🟩 Each premise has type, which can be **Legal** or **Factual.**

🟩 Premises may also include an optional attribute called the Argumentation Scheme. These schemes describe how the premise supports the conclusion. 

👉 **Data Annotation:**

The paper enrich dataset by adding additional annotation and determine the relation between the argumentative statements.

![image.png](image%202.png)

👉 **Data Preparation:** 

The number of possible argumentative pairs in a document grows very fast, especially for the negative pairs 👉 This causes imbalance dataset means most pairs will have no argumentative link and a high computational cost. 

💡 To handle this, the authors implement a **spatial distance strategy**. ****➡️ considering only pairs which their distance is below a certain threshold. This reduces the number of negative pairs while keeping the pairs that are most likely to contain real argumentative links.

Considering two threshold:

- $W_{small}$: Includes pairs within the range of $[-3, 7]$. This range includes 77% of the links in the training sets.
- $W_{large}$:  Includes pairs within the range $[-6, 14]$. This range includes 90% of the links in the training sets.

✨ Note : To evaluate the impact of using distance thresholds, models were tested also on original test set.

👉 **Oversampling:**

It is a technique used in ML to deal with datasets where one class is much smaller than the others (class imbalance).

Oversampling addresses this by:

1. Finding the small class
2. Duplicating the rare samples to increase their number in the training set

This makes the training set more balanced.

![image.png](image%203.png)

🟢 In the paper, the authors oversample positive argumentative pairs in the training set by a 9 and 17 factor in the $𝑊_{𝑠𝑚𝑎𝑙𝑙}$ and $𝑊_{𝑙𝑎𝑟𝑔𝑒}$ training settings, respectively.

👉 **Splitting:**

For data splitting, they applied **document-level random split**, meaning that each document is assigned to either the training, validation, or test set randomly, and all text segments from that document stay in the same split.

---

### 🧐 Models:

🌟 The main task is a binary classification problem to determine whether a link exists between two inputs, source and target, extracted from the same document.

The authors used four models:

1. **Uniform Random Classifier** (baseline)
    
    ➡️ Assigns a link (positive) or no-link (negative) prediction **randomly** to each argumentative pair.
    
2. **Majority Classifier** (baseline)
    
    ➡️ Predicts the **majority class** (no-link) for every argumentative pair.
    

💡 Note: They are non-learning algorithms. They are used to understand whether the real models actually learn something useful.

1. **ResAttArg Ensemble** (Residual Attention Networks)

    ![image.png](image%204.png)
    
    1. Embedding
        - Before the inputs hit the first layer, the sentences are converted into numerical vector representations. The model uses **300-dimensional GloVe pre-trained embeddings** for text representation.
        - The distance between the Source and the Target is encoded as a **10-bit array**.
    2. Dense Layers
        - These are standard fully connected layers. They perform initial transformations on the input to extract relevant patterns for the classification task.
    3. BiLSTM (Bidirectional Long Short-Term Memory)
        - LSTMs are specialized RNNs used to process sequences (like sentences). A **BiLSTM** processes the input sequence in **both forward and backward directions**. This allows the model to capture context from both the words preceding (forward pass) and the words following (backward pass), ➡️ causes a better understanding of the entire sentence's meaning.
    4. Co-Attention
        - This is critical for linking. It allows the model to simultaneously pay attention to **related words or concepts** in both the source sentence and the target sentence. 💥 It finds the dependencies between the two inputs, which is exactly what an argumentative link prediction model needs to do.
    5. Residual Connections
        - Also known as **skip connections**. These connections allow the input from a layer to be passed forward and added directly to the output of a subsequent layer. This mechanism helps to prevent the **vanishing gradient problem** during training, enabling the network to be much deeper and allowing it to train more effectively.
    6. Classification Heads
        - This is the final layer that processed features and maps them to the final outputs. Since ResAttArg is a **multi-task network**, it produces four outputs:
            
            1️⃣ Source Classification: Predicts the argumentative type of the Target (e.g., premise or conclusion)
            
            2️⃣ Target Classification: Predicts the argumentative type of the Target (e.g., premise or conclusion)
            
            3️⃣ Link Prediction: Predicts whether a link exists from Source to Target (binary classification).
            
            4️⃣ Relationship Classification: Predicts the link's type (e.g., SUP, ATT).
            
1. **DistilRoBERTa** (Transformer-based)
    
    It is a **distilled version** of RoBERTa, which itself is a variant of the BERT model.
    
    This process makes the model smaller and faster by:
    
    - reducing the model size by up to **40%**.
    - making it faster by **60%**.
    - retaining **97%** of its language understanding capabilities.
    
    The fundamental mechanism to learn language is the **Masked Language Modelling** (MLM).
    
    👉 Training process: 
    
    🟡 The model is given a sentence, but some random words (tokens) is hidden or **masked.** For RoBERTa, is applied with a **15%** masking probability.
    
    🟡 The model is then trained to predict the **unmasked** word using the surrounding words as context. ➡️ This pre-training allows the model to learn the semantic and syntactic properties of the language.
    
    🟡 After pre-training, the model can be **fine-tuned** to address specific tasks in the same language, such as argument link prediction.
    

---

### 👀 Results:

![image.png](image%205.png)

🟥 The ResAttArg Ensemble shows the best results on the link prediction task. It achieved a maximum F1 score of **0.41** on the positive class (link) and a macro average of **0.70**. 

🟥 The distance constraint does not play a crucial role in training, ➡️ showing robustness of the approach.

🟥 The  DistilRoBERTa model performed poorly without intervention. using of oversampling generally improved DistilRoBERTa's performance but significantly degraded the performance of the ResAttArg Ensemble.

🟥 For secondary tasks → The residual model slightly improved the macro F1 score for **component classification** (distinguishing premises and conclusions) compared to prior work.

🟥 For secondary tasks → The models were unsuccessful at the task of **relation classification** (predicting the type of link), always predicting the majority class (SUP).
