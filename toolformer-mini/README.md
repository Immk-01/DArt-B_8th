# Mini Toolformer Implementation

A small-scale implementation study of  
**Toolformer: Language Models Can Teach Themselves to Use Tools**

## Project Goal

The goal of this project is not to reproduce the full-scale
Toolformer experiments.

Instead, the objective is to learn how to:

1. Read and understand an LLM research paper
2. Convert the proposed method into an algorithm
3. Implement the core pipeline
4. Fine-tune a language model
5. Evaluate whether tool usage actually improves performance

The project focuses on a Calculator Tool as a simplified
implementation of the Toolformer framework.

---

## Original Toolformer Pipeline

```text
Raw Text
   ↓
Generate API Call Candidates
   ↓
Execute API Calls
   ↓
Loss-based Filtering
   ↓
Augmented Dataset
   ↓
Fine-tuning
   ↓
Tool-Using Language Model
