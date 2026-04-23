# CortexFlow Architecture & Brain Logic

This document outlines how CortexFlow processes data, distributes the computational "brain" workload, and precisely calculates cognitive markers from speech.

## High-Level Pipeline

The system is cleanly separated into two distinct stages:
1. **Frontend (Audio & Transcription)**
2. **Backend (Deterministic Cognitive Analysis)**

### 1. Frontend: The Sensory Layer
**Responsibility:** Data Collection & Transcription
- **What it does:** Records user audio and sends it to the `transcribe` API endpoint.
- **The "Brain" usage:** It utilizes the **Gemini API** as its transcription engine. It asks Gemini to transcribe the audio into English and attempt to extract word-level timestamps. 
- **Limitations:** Because Gemini is an LLM and not a dedicated ASR (like Whisper), the timestamps are often hallucinated or empty. The frontend falls back gracefully by calculating pauses where possible and passing the `transcript` and `pause_map` to the backend.

### 2. Backend: The Cognitive Engine
**Responsibility:** Feature Extraction & Clinical Scoring
- **What it does:** This is where the core "brain" of the project lives. It receives the raw text and timing data and runs it through a highly deterministic, mathematical pipeline. 
- **The "Brain" usage:** The backend explicitly **does not use AI to calculate scores**. All cognitive load scoring is done deterministically to prevent hallucinations and ensure clinical reliability. It maps language features directly to brain regions.

---

## How Calculations Are Done (The 5 Domains)

The backend analyzes the speech transcript across five domains, mapping each to a specific neurological region.

### 1. Lexical Domain (Broca's Area) - 22% Weight
Evaluates vocabulary richness and word retrieval.
- **TTR (Type-Token Ratio):** Measures the number of unique words against total words.
- **Lexical Density:** Ratio of "content words" vs. "stop words".
- **Filler Rate:** Counts occurrences of words like "um", "uh", "like", "literally".

### 2. Semantic Domain (Wernicke's Area) - 23% Weight
Evaluates meaning, coherence, and tangential thought.
- **Coherence Index:** Calculates Jaccard similarity between adjacent sentences to see if ideas connect logically.
- **Idea Density:** The average number of content words per sentence.
- **Tangentiality:** Measures how far the speaker drifts from the core semantic thread.

### 3. Prosody Domain (SMA) - 18% Weight
Evaluates speech rhythm and motor timing.
- **Speech Rate (WPM):** Words per minute, estimated using text length and the `pause_map`.
- **Pause Frequency:** How often the speaker pauses for extended periods.
- **Hesitation Ratio:** Ratio of long pauses vs. short grammatical pauses.

### 4. Syntax Domain (DLPFC) - 22% Weight
Evaluates grammatical complexity and executive function.
- **MLU (Mean Length of Utterance):** Average words per sentence.
- **Clause Depth:** Counts subordinating conjunctions ("because", "although") and commas to measure sentence nesting.
- **Passive Ratio:** Frequency of passive voice usage.

### 5. Affective Domain (Amygdala) - 15% Weight
Evaluates emotional state and arousal.
- **Valence:** Ratio of positive words to negative words.
- **Arousal:** Frequency of high-intensity words ("panic", "urgent").
- **Certainty:** Measures "hedge" words ("maybe", "perhaps") to determine speaker confidence.

---

## Final Synthesis: Where AI is Actually Used

After the backend calculates the strict, math-based cognitive load (e.g., `0.74 High Risk`), it uses AI one final time.

1. **Reasoning Model (`gemini-2.0-flash`):** Takes the mathematical scores and writes a human-readable 2-3 sentence summary.
2. **Safety Model (`gemini-2.0-flash`):** Edits the summary to ensure it is strictly non-alarmist and clearly states that it is not a medical diagnosis.

### Summary of "Brain" Workload
- **Gemini AI (20%):** Transcribing audio and writing the final safe summary.
- **Deterministic Math (80%):** Counting, measuring, scoring, and weighting linguistic biomarkers.
