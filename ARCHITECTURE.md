# SnapChef - Architecture Overview

## High-Level System Diagram

```
┌─────────────────────────────────────────────────────────┐
│                  USER INTERFACE                         │
│              (Google Colab Notebook)                    │
│                                                         │
│  • File upload widget (fridge image)                   │
│  • Text input (meal constraints)                       │
│  • HTML cards display (3 recipe options)               │
│  • Interactive selection (1, 2, or 3)                  │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│                   AGENT CORE                            │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │  PLANNER                                         │  │
│  │  • Analyze user constraints                      │  │
│  │  • Plan 3 recipe generation strategies:          │  │
│  │    - Option 1: Fastest prep time                 │  │
│  │    - Option 2: Highest ingredient usage          │  │
│  │    - Option 3: Fewest missing items              │  │
│  └─────────────────┬────────────────────────────────┘  │
│                    │                                    │
│                    ▼                                    │
│  ┌──────────────────────────────────────────────────┐  │
│  │  EXECUTOR                                        │  │
│  │  1. Vision API call (image → ingredients)       │  │
│  │  2. Generation API call (ingredients + prompt   │  │
│  │     → 3 recipes as JSON)                        │  │
│  │  3. Tool call (Unsplash for images)             │  │
│  │  4. Display results to user                     │  │
│  └─────────────────┬────────────────────────────────┘  │
│                    │                                    │
│                    ▼                                    │
│  ┌──────────────────────────────────────────────────┐  │
│  │  MEMORY (Session-based)                         │  │
│  │  • uploaded_image: PIL.Image                    │  │
│  │  • detected_ingredients: String                 │  │
│  │  • generated_recipes: List[Dict]                │  │
│  │  • selected_recipe: Dict                        │  │
│  │  [No persistent storage]                        │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│                  TOOLS / APIs                           │
│                                                         │
│  ┌────────────────────┐  ┌────────────────────┐       │
│  │  Gemini 2.5 Flash  │  │  Unsplash Source   │       │
│  │  • Vision API      │  │  • Recipe images   │       │
│  │  • Generation API  │  │  • No auth needed  │       │
│  └────────────────────┘  └────────────────────┘       │
│                                                         │
│  ┌────────────────────┐                                │
│  │  SMTP (stdlib)     │                                │
│  │  • Email export    │                                │
│  │  • Gmail/Outlook   │                                │
│  └────────────────────┘                                │
└─────────────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│                  OBSERVABILITY                          │
│                                                         │
│  • Console logging (✅ ❌ 🔍 symbols)                   │
│  • Inline output display (ingredients, recipes)        │
│  • Error messages with troubleshooting hints           │
│  • [No persistent logs]                                │
└─────────────────────────────────────────────────────────┘
```

---

## Components

### 1. User Interface

**Platform**: Google Colab Notebook

**Input Methods**:
- File upload widget (provided by `google.colab.files.upload()`)
- Text input via Python `input()` function
- Numeric selection (1, 2, or 3) for recipe choice

**Output Display**:
- Inline image preview (`IPython.display`)
- HTML/CSS recipe cards with images
- Formatted text recipes with Unicode symbols
- Copy/paste shopping list text

**Why Colab**:
- Zero installation required
- Shareable via link
- Built-in GPU access (unused currently but available)
- Perfect for hackathon demos

---

### 2. Agent Core

#### **Planner**

**Location**: Embedded in prompt engineering (Cell 8)

**How It Breaks Down Tasks**:
```python
# Single comprehensive prompt that plans 3 sub-tasks
prompt = f"""
Create 3 DIFFERENT recipe options:

Sub-task 1: Generate fastest recipe (minimize prep/cook time)
Sub-task 2: Generate highest waste-reduction recipe (use most ingredients)
Sub-task 3: Generate budget-friendly recipe (minimize missing items)

Each must respect: {user_constraints}
Use ingredients from: {detected_ingredients}
"""
```

**Planning Strategy**:
- No explicit multi-agent system
- Single Gemini call with comprehensive instructions
- Model internally plans and generates all 3 options
- Diversity enforced through explicit optimization goals

---

#### **Executor**

**Location**: Functions in Cells 6, 8, 10

**LLM Prompt Logic**:
```python
# Vision prompt
vision_prompt = """
Identify ALL visible ingredients.
Organize by category (proteins, vegetables, dairy).
Estimate quantities.
Note freshness (e.g., 'wilted spinach').
"""

# Recipe generation prompt
recipe_prompt = """
Create 3 recipes using available ingredients.
Exclude basic staples (salt, oil, water) from shopping list.
Return as JSON with schema: {...}
"""
```

**Tool-Calling Logic**:
```python
# Step 1: Vision analysis
response = gemini_model.generate_content([vision_prompt, image])
ingredients = response.text

# Step 2: Recipe generation
response = gemini_model.generate_content(recipe_prompt)
recipes = json.loads(response.text)

# Step 3: Image fetching
for recipe in recipes:
    recipe['image_url'] = fetch_unsplash_image(recipe['name'])

# Step 4: Display
display_html_cards(recipes)
```

**Execution Flow**:
1. Call Gemini Vision API (multimodal)
2. Parse text response into ingredient list
3. Call Gemini Generation API with ingredients + constraints
4. Parse JSON response
5. Call Unsplash API for each recipe image
6. Render HTML cards
7. Wait for user selection
8. Display full recipe + shopping list

---

#### **Memory**

**Type**: Session-based in-memory storage

**Storage Mechanism**: Python global variables

**What's Stored**:
```python
# No database, no vector store, no persistent files
uploaded_image = PIL.Image          # Current session image
detected_ingredients = str          # Ingredient list from vision
generated_recipes = List[Dict]      # 3 recipe options
selected_recipe = Dict              # User's choice
```

**Scope**: Single notebook kernel session

**Persistence**: None - all data lost on kernel restart

**Why This Approach**:
- Simplicity for hackathon demo
- No authentication/user management needed
- No database setup required
- Privacy-friendly (no user data stored)

**Future Enhancement**:
- Local JSON file cache
- Cloud database (Firestore/MongoDB)
- User profile system with preferences

---

### 3. Tools / APIs

#### **Google Gemini API**

**Model**: `gemini-2.5-flash`

**Capabilities Used**:
- Multimodal vision (image + text input)
- Text generation with structured output
- JSON schema enforcement

**API Calls**:
```python
import google.generativeai as genai

# Configuration
genai.configure(api_key=os.environ['GEMINI_API_KEY'])

# Usage
model = genai.GenerativeModel('models/gemini-2.5-flash')
response = model.generate_content([prompt, image])
```

**Endpoints**:
- `generateContent` for both vision and text tasks
- No function calling/tools feature used (yet)

**Rate Limits**: Free tier sufficient for demo usage

---

#### **Unsplash Source API**

**Purpose**: Fetch recipe images for visual appeal

**Implementation**:
```python
def get_recipe_image(search_query):
    url = f"https://source.unsplash.com/400x300/?{search_query},food,dish"
    return url
```

**Authentication**: None required (public API)

**Rate Limits**: None on source API

**Why This Tool**: 
- No API key setup
- High-quality food photography
- Simple URL construction

---

#### **SMTP Email (Python stdlib)**

**Purpose**: Send shopping list via email

**Implementation**:
```python
import smtplib
from email.mime.text import MIMEText

server = smtplib.SMTP('smtp.gmail.com', 587)
server.starttls()
server.login(user_email, password)
server.send_message(msg)
```

**Supported Providers**: Gmail, Outlook, Yahoo (auto-detected)

**Authentication**: User provides credentials at runtime

---

### 4. Observability

#### **Logging**

**Approach**: Console output with emoji indicators

**Log Levels**:
```
✅ Success messages
❌ Error messages
🔍 Processing/analyzing indicators
⚠️  Warning messages
💡 Informational tips
```

**Example Log Flow**:
```
Cell 5: test_image = upload_fridge_image()
Output:
  📤 Click 'Choose Files' to upload...
  ✅ Uploaded: fridge.jpg
  🖼️  Image size: 1024 x 768 pixels

Cell 7: ingredients = detect_ingredients(test_image)
Output:
  🔍 Analyzing image with Gemini Vision...
  ✅ Analysis complete!
  📋 DETECTED INGREDIENTS:
  Proteins: 6 eggs, 1 chicken breast
  [...]
```

**Log Storage**: [Not Available] - Console output only, no persistent logs

---

#### **Error Handling**

**API Failures**:
```python
try:
    response = model.generate_content([prompt, image])
except Exception as e:
    print(f"❌ Error during API call: {e}")
    print("🔍 Troubleshooting tips:")
    print("  1. Check API key validity")
    print("  2. Verify internet connection")
    print("  3. Check Gemini API status")
```

**JSON Parsing Errors**:
```python
try:
    recipes = json.loads(response.text)
except json.JSONDecodeError as e:
    print(f"⚠️  JSON parsing failed: {e}")
    print("Raw response (first 500 chars):")
    print(response.text[:500])
    print("\n💡 Try running the cell again")
```

**Retry Logic**: [Not Available] - User must manually re-run cell

---

#### **Reasoning Step Traceability**

**How Judges Can Follow**:

1. **Vision Step** - Ingredient list printed inline
```
📋 DETECTED INGREDIENTS:
Proteins: 6 eggs, 1 chicken breast
Vegetables: 2 red bell peppers, 1 cup broccoli
Dairy: 1 bottle milk (2/3 full)
```

2. **Planning Step** - User constraint echoed
```
💬 Your constraints: low-carb dinner
```

3. **Generation Step** - Summary of 3 options shown
```
✅ 3 recipe options generated successfully!

Option 1: Chicken Veggie Stir-Fry
  ⏱️  Prep: 25min | 🔥 450 cal | 🛒 2 items
```

4. **Selection Step** - Choice confirmed
```
Which recipe would you like? (1-3): 1
✅ You selected: Chicken Veggie Stir-Fry
```

5. **Final Output** - Complete recipe displayed
```
🍽️  CHICKEN VEGGIE STIR-FRY
[Full recipe details...]
```

---

## Data Flow Diagram

```
User Action          Data                    Processing
───────────         ──────                  ──────────

[Upload Image] ──> PIL.Image ────────────> Gemini Vision API
                                                 │
                                                 ▼
                                          Ingredient List
                                            (text string)
                                                 │
[Enter Text] ───> "low-carb dinner" ───────────>│
                                                 │
                                                 ▼
                                        Gemini Generation API
                                                 │
                                                 ▼
                                        JSON (3 recipes)
                                                 │
                                                 ▼
                                          Unsplash URLs
                                                 │
                                                 ▼
                                        HTML Card Display
                                                 │
[Select "2"] ───> Integer ──────────────────────│
                                                 ▼
                                        Recipe Dictionary
                                                 │
                                                 ▼
                                        Text Formatting
                                                 │
                                                 ▼
                        ┌───────────────────────┴────────────────────┐
                        ▼                                            ▼
                [Email Option]                             [Copy/Paste Option]
                    SMTP                                   Formatted String
```

---

## Key Design Decisions

### **1. Single Model vs. Multi-Agent**
**Decision**: Use one Gemini model for all tasks  
**Why**: Simpler, faster, lower latency than multiple agents communicating  
**Tradeoff**: Less modularity, but sufficient for current scope

### **2. Session-Only Memory**
**Decision**: No persistent storage  
**Why**: Faster development, no auth needed, privacy-friendly  
**Tradeoff**: No personalization or learning across sessions

### **3. Colab vs. Web App**
**Decision**: Google Colab notebook  
**Why**: Zero setup, easy sharing, perfect for demos  
**Tradeoff**: Not production-ready, requires manual cell execution

### **4. Flash vs. Pro Model**
**Decision**: Gemini 2.5 Flash  
**Why**: 10x cheaper, 3x faster, quality sufficient for task  
**Tradeoff**: Slightly less creative recipe suggestions

### **5. Structured JSON Output**
**Decision**: Enforce strict JSON schema in prompt  
**Why**: Reliable parsing, predictable structure  
**Tradeoff**: Less flexibility if schema needs change

---

## Scalability Considerations

**Current**: Single-user, synchronous, session-based

**Future Enhancements**:
- Async API calls for faster response
- Caching for common ingredient combinations
- User authentication and profiles
- Cloud deployment (Cloud Run, App Engine)
- Database for persistent memory (Firestore)

**Estimated Capacity**:
- Current: 1 user at a time
- Gemini free tier: ~60 requests/minute
- Bottleneck: Sequential API calls (10-15 sec per workflow)
