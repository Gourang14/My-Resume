# Physician Notetaker: Clinical NLP Pipeline

## Project Overview
This project is an AI-driven medical documentation system designed to transform unstructured physician–patient conversations into structured, actionable clinical insights. Developed for the **Emitrr AI Engineer Intern assessment**, the pipeline performs **Medical Named Entity Recognition (NER)**, **Sentiment & Intent Analysis** and **Automated SOAP Note Generation**, with a strong emphasis on **clinical safety, explainability and zero-inference extraction**.

---

## Key Features
- **Hybrid Medical NER:** Combines spaCy’s statistical model with generalized rule-based patterns for robust and recall-aware entity extraction.
- **Transformer-based Sentiment Analysis:** Uses zero-shot classification to detect patient sentiment and intent without task-specific fine-tuning.
- **Modular SOAP Generation:** Converts extracted information into a structured Subjective–Objective–Assessment–Plan format.
- **Safety-First Design:** Enforces negation detection and verbatim extraction to prevent hallucinations or unsafe medical inference.

---

## 🛠️ Technical Design & Philosophy

### 1. Medical NER (Information Extraction)
The NER pipeline follows a **hybrid architecture** using `spaCy (en_core_web_sm)` and an `EntityRuler`.

- **Clinical Safety vs. Recall:**  
  The system uses **generalized patterns** (e.g., numeric expressions followed by “sessions”) to capture treatments like *physiotherapy* without hard-coding transcript-specific phrases. This balances recall while remaining robust across varied conversations.

- **Zero-Inference Policy:**  
  All extracted entities are returned **verbatim from the transcript**. The system intentionally avoids semantic normalization or reinterpretation (e.g., converting “hit my head” into “head impact”) to preserve original clinical context.

- **Negation Handling:**  
  A heuristic negation window checks for cues such as *“no”*, *“denies”*, or *“without”* to ensure that mentioned but absent symptoms are not falsely reported.

---

### 2. Sentiment & Intent Analysis
Patient sentiment and intent are analyzed using **zero-shot classification** with the `valhalla/distilbart-mnli-12-1` transformer model.

- This approach enables detection of nuanced emotional states such as **Anxious**, **Neutral** or **Reassured**, as well as intents like **Seeking Reassurance** or **Reporting Symptoms**, without requiring domain-specific labeled data.

---

### 3. SOAP Note Generation
The `SOAPGenerator` maps extracted entities into a clinically meaningful structure:

- **Subjective:** Patient-reported symptoms and emotional state.
- **Objective:** Clinician-observed findings (left empty if not explicitly documented in the transcript).
- **Assessment:** Diagnosis clearly marked as *patient-reported* where applicable.
- **Plan:** Explicitly stated treatments, with logical separation between **Medications** and **Therapies**.

This strict separation ensures clinical readability and avoids unsafe inference.

---

## Technical Questions & Answers

### I. Medical NLP Summarization

**Q: How would you handle ambiguous or missing medical data?**  
**A:** In future iterations, this system could be extended with a **confidence-aware extraction layer**, where low-confidence entities are flagged for physician review instead of being auto-included. Missing relationships (e.g., a symptom without an anatomical location) could be identified using relation extraction techniques and highlighted for clarification.

**Q: What pre-trained NLP models would you use for medical summarization?**  
**A:** For domain-specific summarization, models such as **BioBART** or **Clinical-T5** would be strong candidates due to their pre-training on PubMed and clinical datasets like MIMIC.

---

### II. Sentiment & Intent Analysis

**Q: How would you fine-tune BERT for medical sentiment detection?**  
**A:** I would start with **BioBERT**, add a classification head, and fine-tune it on labeled patient–doctor dialogue data using a low learning rate to preserve clinical language understanding.

**Q: What datasets would you use for training a healthcare-specific sentiment model?**  
**A:** Suitable datasets include **MIMIC-IV**, **MTSamples**, **MedDialog** and **SMM4H**, which capture both formal clinical language and informal patient expressions.

---

### III. SOAP Note Generation (Bonus)

**Q: How would you train an NLP model to map medical transcripts into SOAP format?**  
**A:** This problem can be modeled as a **sequence-to-sequence task**. Architectures such as **LED (Long Encoder–Decoder)** are well-suited for long clinical conversations and could be fine-tuned on paired transcript-SOAP datasets (e.g., MT-SOAP), subject to clinical validation.

**Q: What rule-based or deep-learning techniques would improve the accuracy of SOAP note generation?**  
**A:** A hybrid strategy is most effective.  
- **Rule-based techniques** such as speaker attribution (Patient vs. Physician), section-specific keyword rules, and negation detection help ensure correct placement of information into Subjective, Objective, Assessment, and Plan sections while maintaining clinical safety.  
- **Deep-learning techniques** could include fine-tuned sequence-to-sequence models that learn section boundaries implicitly, as well as attention mechanisms to associate symptoms with diagnoses or treatments. In practice, combining rule-based safeguards with learned models provides better accuracy and explainability than using either approach alone.

---

## Project Structure
```text
Gourang_Agarwal_Emitrr_Assignment/
├── data/
│   └── transcript.txt
├── notebook/
│   ├── demo.ipynb
│   └── verify_pipeline.py
├── outputs/
│   ├── medical_summary.json
│   ├── sentiment_intent.json
│   └── soap_note.json
├── src/
│   ├── preprocess.py
│   ├── ner_extraction.py
│   ├── keyword_extraction.py
│   ├── summarizer.py
│   ├── sentiment_intent.py
│   └── soap_generator.py
├── README.md
└── requirements.txt

---

## Setup & Execution

### 1. Environment Setup
Create a virtual environment and install dependencies:

```bash
python -m venv venv
source venv/bin/activate      # On Windows: venv\Scripts\activate
pip install -r requirements.txt

### 2. Model Setup

Download the spaCy English model used for Medical NER:

```bash
python -m spacy download en_core_web_sm

### 3. Run the Pipeline

Open and execute all cells in the demo notebook:

```text
notebook/demo.ipynb

### This will:
- Load the transcript
- Run preprocessing
- Perform Medical NER
- Extract keywords
- Analyze sentiment and intent
- Generate the SOAP note
- Save outputs to the `outputs/` directory


### 📄 Generated Outputs

After execution, the following files are created in the `outputs/` directory:

- **medical_summary.json** — Structured medical summary extracted from the transcript  
- **sentiment_intent.json** — Patient sentiment and intent analysis  
- **soap_note.json** — Automated SOAP note generated from extracted entities  

All outputs are **fully grounded in the transcript** and contain **no inferred or hallucinated medical information**.

