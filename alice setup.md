# Alice Setup & Architecture Guide 🐰

## 1. Overview
Alice is a **Hybrid AI Assistant** designed for the *Goddess of Victory: Nikke* community. Unlike standard LLM chatbots, Alice uses a **"Brain First, LLM Second"** architecture. She prioritizes hard-coded game data and deterministic logic for factual queries, using the LLM (Llama 3.2) primarily for persona generation and conversational fallback.

---

## 2. File Structure (`utils/alice/`)

Alice's "brain" consists of a core Python script and a suite of JSON data files.

| File Name | Purpose |
|-----------|---------|
| **`brain.py`** | **The Core.** Connects all modules, handles logic, routing, and response generation. |
| **`knowledge.json`** | **Q&A Database.** Stores specific game mechanics (e.g., "What is Overload gear?"). |
| **`nikke_characters.json`** | **Character Data.** Stats, skills, bursts, and weapon types for all Nikkes. |
| **`nikke_meta.json`** | **Meta Data.** Tier lists, investment guides, and cube recommendations. |
| **`solo_raid_teams.json`** | **Raid Teams.** Pre-defined teams for Solo Raid bosses (e.g., Mother Whale). |
| **`elemental_teams.json`** | **Elemental Teams.** Teams optimized for specific elemental weaknesses. |
| **`cubes.json`** | **Harmony Cubes.** Data on cube effects and recommended users. |
| **`feature_aliases.json`** | **Search Aliases.** Maps casual terms (e.g., "nuke", "healer") to game mechanics. |
| **`intent_classifier.py`** | **ML Module.** A Scikit-Learn model that predicts user intent (GAME vs CHAT). |
| **`image_utils.py`** | **Visuals.** Handles image processing and fetching for character portraits. |
| **`database.py`** | **Long-Term Memory.** Connects to the PostgreSQL database for user logs. |

---

## 3. How `brain.py` Connects Everything

`brain.py` is the central nervous system. It follows the **Singleton Pattern**, ensuring only one instance of Alice exists in memory.

### **Initialization Process (`__init__`)**
1.  **Loads JSONs**: Ingests all data files (`knowledge.json`, `nikke_characters.json`, etc.) into memory.
2.  **Builds Indices**: Creates lookup maps for fast searching (e.g., mapping "Doro" -> "Dorothy").
3.  **Trains/Loads ML**: Initializes the `IntentClassifier` (Scikit-Learn) to understand natural language.
4.  **Connects to DB**: Establishes a link to PostgreSQL for logging and memory.

### **The Thinking Process (`think()`)**
Every user message goes through the `think()` pipeline:
1.  **Input**: User sends a message.
2.  **Intent Classification**: Alice determines *what* the user wants (GAME, CHAT, or SEARCH).
3.  **Routing**:
    *   **GAME**: Queries internal JSON data (Deterministic).
    *   **SEARCH**: Googles the web (DuckDuckGo).
    *   **CHAT**: Asks the LLM (Llama 3.2).
4.  **Response**: The raw data is wrapped in Alice's persona and sent back.

---

## 4. The Trigger System (Reflexes)

Alice uses a **4-Layer Unified Decision Pipeline** to decide how to respond. This prevents the LLM from hallucinating game facts.

### **Layer 1: Hard-Coded Entity Match (The "Red Hood" Rule)**
*   **Logic**: If a known Character Name (e.g., "Red Hood") or Boss Name (e.g., "Chatterbox") is found in the sentence.
*   **Action**: IMMEDIATELY classifies as **GAME**.
*   **Why**: Ensures 100% reliability for character queries.
*   **Exception**: If the name is "Alice" (herself), it checks for context (e.g., "Alice build" = GAME, "Alice you are cute" = CHAT).

### **Layer 2: Keyword Triggers (Configuration)**
*   **Logic**: Checks for specific keywords defined in `TRIGGER_CONFIG` (inside `brain.py`).
*   **Examples**: "tier list", "meta", "cube", "skill", "burst".
*   **Action**: Classifies as **GAME** or **SEARCH**.

### **Layer 3: Machine Learning (The Intuition)**
*   **Logic**: If no hard rules match, the `IntentClassifier` (Scikit-Learn) predicts the intent based on training data.
*   **Example**: "Who is the best healer?" -> **GAME** (implied context).

### **Layer 4: Fallback (Chat)**
*   **Logic**: If all else fails, it's treated as casual conversation.
*   **Action**: Routes to Llama 3.2 with the system prompt "You are Alice...".

---

## 5. Hard-Coding + AI (The Hybrid Model)

Alice's "Magic" comes from mixing strict code with flexible AI.

### **"Brain is Truth" (Hard-Coding)**
*   **Stats/Skills**: Never guessed. Pulled directly from `nikke_characters.json`.
*   **Math**: Solved by Python (`eval`), not the LLM.
*   **Dates/Events**: Hard-coded to prevent "I don't know the date" errors.

### **"Persona Wrapper" (AI)**
*   Once the hard data is found (e.g., "Red Hood is Tier SS"), it is passed to the LLM.
*   **Prompt**: "Here is the data: Red Hood is SS Tier. Rewrite this as Alice from Nikke."
*   **Result**: "Red Hood is super strong, Rabbity! She's definitely SS Tier! *sparkles*"

This ensures **Accuracy** (from Code) and **Personality** (from AI).
