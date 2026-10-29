# multi agent individual prompts llm

##  Assignment Description

Clarity classification of responses in political interviews (Clear Reply, Ambivalent, Clear Non-Reply), based on SemEval 2026 Task 6 (CLARITY),
using a lightweight multi-agent prompting framework, where the classification task is split across four specialized agents, each with its own 
individual prompt. All agents run on the instruction-tuned language model Qwen3.5-0.8B from the Hugging Face Transformers library, without any 
fine tuning.

## Overview

Instead of using a single prompt to classify the answer in one step, the task is split across **four agents**.
**Each agent has its own dedicated prompt** and handles one specific part of the problem. All agents run on the same model, **Qwen3.5-0.8B**, 
with no fine tuning. Their behavior comes entirely from their individual prompts

### The agents

**1. Question Intent Agent**
- **Runs:** first
- **Input:** the question only (it is the first agent, so it receives nothing from other agents).
- **What it does:** identifies what the question is asking for and what a clear answer would need to contain
- **Output:**
  - the type of question, e.g. yes/no, why, how, stance, action plan (`question_type`)
  - a one sentence description of what the question asks (`question_intent`)
  - a list of specific points the answer must address (`must_address`)
  - what a clear answer must contain (`clear_answer_criteria`)
  - the minimum information needed for a Clear Reply (`minimal_sufficient_answer`)

**2. Answer Content Agent**
- **Runs:** second, after the Question Intent Agent has finished
- **Input:** the question and the answer (it does not use the Question Intent Agent's output, it analyzes the answer independently)
- **What it does:** extracts what the answer actually says about the question: direct answers, concrete facts, vague statements, topic shifts and refusals
- **Output:**
  - a one sentence summary of the parts of the answer relevant to the question (`relevant_answer_summary`)
  - whether the answer contains a direct reply such as "yes", "no" or "I did" and the exact phrase (`direct_answer_present`, `direct_answer_text`)
  - concrete facts, actions, reasons or commitments in the answer (`concrete_information`)
  - relevant but vague or general statements (`vague_or_general_information`)
  - content that changes the topic or answers a different question (`off_target_content`)
  - signs of refusing or avoiding the question (`refusal_or_non_answer_signals`)

**3. Gap and Evasion Agent**
- **Runs:** third, after the Answer Content Agent has finished
- **Input:** the question, the answer, **plus the outputs of the Question Intent Agent and the Answer Content Agent**
- **What it does:** compares what the question requires (from the Question Intent Agent) with what the answer provides (from the Answer Content 
    Agent). It detects missing information, partial answers and evasion patterns such as dodging, deflection or declining to answer
- **Output:**
  - how much of the question the answer covers: complete, mostly complete, partial, minimal or none (`coverage`)
  - how serious the gap is: none, minor, partial or major (`gap_level`)
  - the type of answer or evasion, e.g. explicit answer, dodging, deflection, declining to answer (`best_evasion_category`)
  - the points from the question that the answer did not address (`missing_requirements`)
  - short evidence from the answer showing it answers and showing it evades (`evidence_for_answering`, `evidence_for_evasion_or_gap`)
  - a one sentence explanation of the gap (`gap_explanation`)
  - an overall verdict: answers, partially answers or does not answer (`overall_verdict`)

**4. Decision Agent**
- **Runs:** fourth (last), after the Gap and Evasion Agent has finished
- **Input:** the question, the answer, **plus the outputs of all three previous agents**
- **What it does:** summarizes the key signals from the previous agents and uses them, together with label definitions, decision rules 
    and five few-shot examples, to assign the final label
- **Output:**
  - a summary of the key signals it used: coverage, gap level and overall verdict (`key_signals_used`)
  - the final label: Clear Reply, Ambivalent or Clear Non-Reply (`label`)
  - a confidence score for the label (`confidence`)
  - a one sentence reason for the decision (`main_reason`)
  - which decision rule determined the label (`used_rule`)

### Pipeline

The four agents run **sequentially** (one after another) for each question-answer pair:

```
Question + Answer
       │
       ▼
1. Question Intent Agent
       │
       ▼
2. Answer Content Agent
       │
       ▼
3. Gap and Evasion Agent
       │
       ▼
4. Decision Agent
       │
       ▼
Clear Reply / Ambivalent / Clear Non-Reply
```

Each agent returns structured **JSON**. The Question Intent Agent works only on the question, while the Answer Content Agent works on the 
question and the answer. The Gap and Evasion Agent receives both of their outputs and the Decision Agent receives the outputs of all three 
previous agents. Later agents therefore reason over clear, structured information instead of free text

##  Dataset

The dataset used in this project is available on Hugging Face: [ailsntua/QEvasion](https://huggingface.co/datasets/ailsntua/QEvasion)

Backup copy of the dataset, in case the above original link is no longer available: [EmilBa122/QEvasion-backup](https://huggingface.co/datasets/EmilBa122/QEvasion-backup)
