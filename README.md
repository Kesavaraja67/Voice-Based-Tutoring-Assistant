# 🚀 Adaptive Voice Tutor for Pediatric Learners

[![Python Version](https://img.shields.io/badge/python-3.9%2B-blue)](https://www.python.org/)
[![Flask Version](https://img.shields.io/badge/flask-2.x-green)](https://flask.palletsprojects.com/)
[![PyTorch Version](https://img.shields.io/badge/pytorch-2.x-red)](https://pytorch.org/)
[![Hugging Face Transformers](https://img.shields.io/badge/transformers-4.x-orange)](https://huggingface.co/docs/transformers/index)
[![License](https://img.shields.io/badge/license-MIT-lightgrey)](LICENSE)

---

## 💡 Project Overview

This repository showcases an innovative **Adaptive Voice Tutor Prototype** specifically designed to engage and assist young learners (ages 5-10) through natural speech interaction. The project tackles critical challenges in developing conversational agents for children, primarily focusing on **mitigating low-resource bias** in data and handling the **high acoustic variability of child speech**.

Our solution proposes and implements a novel **hybrid Machine Learning architecture** that judiciously combines specialized, lightweight models for core classification tasks with a powerful Large Language Model (LLM) for adaptive, context-aware dialogue generation.

---

## 🎯 The Core Problem & Our Solution

**Problem Statement:**
Developing effective voice-based conversational tutors for pediatric users presents two significant hurdles:
1.  **Acoustic Variability:** Child speech patterns (e.g., higher pitch, varied speech rates, less distinct phoneme articulation) often lead to poor performance in standard Automatic Speech Recognition (ASR) systems trained predominantly on adult voices.
2.  **Low-Resource Bias:** There is a severe scarcity of publicly available, large-scale, annotated speech and intent datasets tailored specifically for children and educational contexts. This limits the applicability of data-hungry deep learning models.

**Our Hybrid Solution:**
We address these challenges by implementing a hybrid ASR-NLP system that strategically offloads complex tasks:
* **Local, Optimized Models:** Lightweight, custom-trained models handle high-accuracy, domain-specific classification (e.g., simple commands, core intents) on limited data.
* **External LLM Integration:** A robust general-purpose LLM (Gemini API) is employed for generating dynamic, pedagogically sound, and contextually adaptive conversational responses, guided by the precise classifications from our local models.

---

## 🛠️ Technical Architecture & Methodology

The project's architecture is a testament to leveraging appropriate ML models for specific tasks within a full-stack deployment.

### 1. Machine Learning Pipeline

Our solution comprises two distinct, custom-trained ML models, each addressing a specific challenge:

| Component | Model / Architecture | Task Addressed | Key Contributions / Findings |
| :-------- | :------------------- | :------------- | :---------------------------- |
| **ASR (Voice Command Recognition)** | **1D Convolutional Neural Network (CNN)** on **Mel-Frequency Cepstral Coefficients (MFCCs)**. | Accurately recognizes a limited vocabulary of core commands (e.g., 'yes', 'no', 'up', 'down', 'stop') despite acoustic variability. | Achieved **>80% accuracy** on this targeted task, demonstrating the efficiency of compact CNNs for specific, low-resource ASR challenges. |
| **NLP (Conversational Intent Classification)** | **DistilBERT (Fine-tuned)** for Sequence Classification. | Classifies the user's pedagogical intent (e.g., `NUMERACY`, `LITERACY`, `CONFUSION`, `SHIFT`, `GREETING`) from the ASR's transcribed output. | Quantified **Catastrophic Forgetting (Data Loss)** by observing validation loss divergence when fine-tuning a large pre-trained model on a small (`N=60`) synthetic intent dataset. Mitigation strategies (low learning rate, early stopping) were crucial. |

### 2. Full-Stack Deployment Strategy

The system is engineered as a three-tier architecture to ensure resilience, scalability, and an engaging user experience:

* **Frontend (`index.html`):**
    * Developed with **HTML/JavaScript**.
    * Handles user interaction, microphone access (`MediaRecorder`), 1-second audio capture, Base64 encoding of audio, and dynamic display of transcription and tutor responses.
    * Communicates with the Flask Backend via `fetch` requests.
* **Backend (`server.py`):**
    * A **Python Flask server**.
    * **Core role:** Loads and hosts both the trained **CNN (ASR)** and **DistilBERT (NLP)** models.
    * Provides two RESTful API endpoints: `/transcribe` (for ASR) and `/classify_intent` (for NLP).
    * Orchestrates the sequential processing: Raw Audio (Base64) $\rightarrow$ ASR $\rightarrow$ Transcription $\rightarrow$ NLP $\rightarrow$ Intent.
    * Communicates the classified intent to the external LLM.
* **LLM (External Service):**
    * Utilizes the **Google Gemini API**.
    * Receives the **classified intent** (e.g., "NUMERACY") from the Flask backend.
    * Leverages a carefully crafted **system instruction/prompt** to generate adaptive, friendly, and pedagogically appropriate conversational responses tailored to the child's identified intent.

### System Architecture Diagram

This diagram illustrates the data flow and interaction between the system's components:

```mermaid
graph TD
    %% -- Components --
    A[Frontend (HTML/JS, MediaRecorder)]
    
    subgraph Python Backend (Flask)
        direction LR
        B1(ASR (CNN))
        B2(NLP (DistilBERT))
        B1 --- B2
    end
    
    C[LLM (Gemini API)]

    %% -- Main Data Flow --
    A -- Audio Input (1 sec, Base64) --> B1;
    B1 -- Transcription --> B2;
    B2 -- Classified Intent --> C;
    C -- Tutor Response (Text) --> A;

    %% -- Styling for Clarity --
    style A fill:#BBDEFB,stroke:#1E88E5,stroke-width:2px;
    style B1 fill:#A5D6A7,stroke:#388E3C,stroke-width:2px;
    style B2 fill:#FFCC80,stroke:#FF9800,stroke-width:2px;
    style C fill:#9575CD,stroke:#673AB7,stroke-width:2px;
    
    linkStyle 0 stroke-width:2px,fill:none,stroke:darkblue;
    linkStyle 1 stroke-width:2px,fill:none,stroke:green;
    linkStyle 2 stroke-width:2px,fill:none,stroke:orange;
    linkStyle 3 stroke-width:2px,fill:none,stroke:purple;
```
