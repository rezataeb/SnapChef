# SnapChef - Technical Explanation

## 1. Agent Workflow

SnapChef processes user input through a sequential pipeline with three main stages:

### Step-by-Step Process:

**1. Receive User Input**
- User uploads a fridge/pantry image via Colab file upload widget
- User provides text constraints (e.g., "low-carb dinner", "quick lunch under $10")

**2. Retrieve Relevant Memory**
- [Not Available] - Current implementation uses session-only memory with no retrieval mechanism

**3. Plan Sub-Tasks**
- Agent receives detected ingredients + user constraints
- Internally plans to generate 3 recipe options with different optimization goals:
  - **Sub-task A**: Generate fastest recipe (minimize prep time)
  - **Sub-task B**: Generate highest waste-reduction recipe (maximize ingredient usage)
  - **Sub-task C**: Generate budget-friendly recipe (minimize missing items)
- All sub-tasks executed in a single Gemini API call with comprehensive prompt

**4. Call Tools/APIs**
- **Gemini API**: Vision analysis (multimodal input)
- **Gemini API**: Recipe generation (text generation with JSON schema)
- **Unsplash Source API**: Fetch recipe images based on dish names

**5. Summarize and Return Final Output**
- Parse JSON response containing 3 recipe options
- Display interactive HTML cards with images
- User selects preferred option
- Display full recipe with instructions and shopping list
- Offer export options (email or copy/paste)

---

## 2. Key Modules

### Vision Module (`detect_ingredients()` - Cells 6-7)

**Purpose**: Analyzes fridge images using Gemini's multimodal capabilities

**Key Logic**:
```python
def detect_ingredients(image, additional_context=""):
    model = genai.GenerativeModel('models/gemini-2.5-flash')
    prompt = """
    Analyze this fridge/pantry image.
    Identify ALL visible ingredients with quantities.
    Organize by category.
    """
    response = model.generate_content([prompt, image])
    return response.text
```

**Input**: PIL Image object  
**Output**: Text string with categorized ingredients

---

### Recipe Planner (`generate_recipe_options()` - Cell 8)

**Purpose**: Generates 3 diverse recipe options optimized for different goals

**Key Logic**:
```python
def generate_recipe_options(detected_ingredients, user_constraints):
    prompt = f"""
    Create 3 DIFFERENT recipes using: {detected_ingredients}
    Constraints: {user_constraints}
    Exclude basic staples (salt, oil, water).
    Return as JSON with specific schema.
    """
    response = model.generate_content(prompt)
    return json.loads(clean_response(response.text))
```

**Input**: Ingredient list (string) + constraints (string)  
**Output**: List of 3 recipe dictionaries

---

### Display Executor (`display_recipe_options()`, `display_full_recipe()` - Cells 8, 10)

**Purpose**: Renders recipes in user-friendly HTML/text formats

**Key Features**:
- HTML/CSS cards with Unsplash images
- Color-coded badges (prep time, calories, items needed)
- Formatted text with Unicode symbols
- Interactive user selection handling

---

### Memory Store (Session Variables)

**Type**: In-memory Python variables (no persistent storage)

**Stored State**:
```python
uploaded_image = None           # PIL.Image object
detected_ingredients = None     # String with ingredient list
generated_recipes = None        # List of 3 recipe dicts
selected_recipe = None          # Single recipe dict
```

**Scope**: Single notebook session only  
**Persistence**: [Not Available] - No persistent memory, resets on kernel restart

---

## 3. Tool Integration

### Gemini API (`google.generativeai`)

**Purpose**: Vision analysis and recipe generation

**How We Call It**:
```python
# Configuration
import google.generativeai as genai
genai.configure(api_key=os.environ['GEMINI_API_KEY'])

# Vision call (multimodal)
model = genai.GenerativeModel('models/gemini-2.5-flash')
response = model.generate_content([text_prompt, image])

# Text generation call
response = model.generate_content(text_prompt)
```

**Function**: `model.generate_content()`  
**Endpoints Used**: `generateContent` for both vision and text generation

---

### Unsplash Source API

**Purpose**: Fetch recipe images for visual display

**How We Call It**:
```python
def get_recipe_image(search_query):
    base_url = "https://source.unsplash.com/400x300/"
    return f"{base_url}?{search_query},food,dish"

# Example: get_recipe_image("chicken stir fry")
```

**Function**: Direct URL construction (no SDK)  
**Authentication**: None required (public API)

---

### SMTP Email API

**Purpose**: Send shopping lists via email

**How We Call It**:
```python
import smtplib
from email.mime.text import MIMEText

server = smtplib.SMTP('smtp.gmail.com', 587)
server.starttls()
server.login(sender_email, password)
server.send_message(msg)
```

**Function**: Standard library `smtplib.SMTP()`  
**Authentication**: User provides credentials at runtime

---

## 4. Observability & Testing

### Logging

**Console Output**: All key steps print progress indicators and status messages directly to notebook output.

**Example Log Flow**:
```
📸 Uploading image...
✅ Image uploaded: 1024x768 pixels

🔍 Analyzing image with Gemini Vision...
✅ Analysis complete!

📋 DETECTED INGREDIENTS:
Proteins: 6 eggs, 1 chicken breast
[...]

🔍 Generating 3 recipe options...
✅ 3 recipe options generated successfully!
```

**Log Files**: [Not Available] - No persistent log files, console output only

---

### Error Handling

**API Failures**:
```python
try:
    response = model.generate_content([prompt, image])
except Exception as e:
    print(f"❌ Error: {e}")
    print("🔍 Troubleshooting tips:")
```

**JSON Parsing Failures**:
```python
try:
    recipe_data = json.loads(response_text)
except json.JSONDecodeError as e:
    print(f"⚠️ JSON parsing error: {e}")
    print("Raw response:", response.text[:500])
```

---

### How Judges Can Trace Decisions

**Manual Testing Path**:
1. Run cells 1-3 (setup, see ✅ confirmations)
2. Run cells 4-5 (upload image, see preview)
3. Run cells 6-7 (see detected ingredients printed)
4. Run cell 8 (functions defined)
5. Run cell 9 (see 3 recipe cards, make selection)
6. Run cells 10-11 (see full recipe, test export)

**Traceability**:
- All intermediate results printed inline
- User selections echoed back
- API responses partially displayed
- JSON parsing errors show raw output

---

### Automated Testing

**Test Scripts**: [Not Available] - No `TEST.sh` or automated test suite

**Testing Approach**: Manual execution through sequential notebook cells

---

## 5. Known Limitations

### 1. Long-Running API Calls

**Issue**: Combined vision + recipe generation takes 10-15 seconds

**Breakdown**:
- Vision analysis: 3-5 seconds
- Recipe generation: 5-8 seconds
- Image fetching: 1-2 seconds

**Why**: Sequential API calls to Gemini + Unsplash

**Current Mitigation**: Show progress indicators, use Flash model (faster than Pro)

---

### 2. Handling of Ambiguous User Inputs

**Issue**: Vague constraints like "healthy meal" are subjective

**Examples**:
- "healthy" - Low-cal? High-protein? Whole foods?
- "quick" - 10 minutes? 30 minutes?
- "budget-friendly" - No specific dollar amount

**Impact**: Generated recipes may not match user's actual intent

**Current Mitigation**: AI makes reasonable assumptions, explains in chef_notes

---

### 3. Ingredient Detection Accuracy (~10% Error Rate)

**Issue**: Vision model occasionally misidentifies or misses items

**Why**:
- Blurry photos reduce accuracy
- Small items (spices, condiments) often missed
- Similar-looking items confused

**Current Mitigation**: Prompt instructs model to only include high-confidence detections

---

### 4. No Persistent Memory

**Issue**: Agent has no memory between sessions

**Impact**:
- Can't remember past preferences
- Can't learn from recipe ratings
- Repeats same questions every session

**Why**: Session-only architecture (no database, no auth system)

---

### 5. Fixed Serving Size (2 Servings)

**Issue**: All recipes generated for exactly 2 servings

**Impact**: Singles and families must manually scale quantities

**Why**: Scaling logic not implemented in current version

---

### 6. JSON Parsing Fragility

**Issue**: Gemini occasionally returns malformed JSON

**Frequency**: ~2-5% of requests

**Current Mitigation**: Strip markdown blocks, show error with raw response, user can retry

---

### 7. No Real-Time Pricing Data

**Issue**: Shopping lists show items but not actual current prices

**Impact**: User doesn't know total cost until they shop

**Status**: Price lookup via Google Search planned for future version

---

### 8. English-Only Interface

**Issue**: All prompts, recipes, and UI text in English only

**Impact**: Excludes non-English speakers

**Why**: Multilingual support requires translated prompts, different measurement systems, localized food names

---

## Performance Notes

**Typical Session Cost**: ~$0.005 per complete workflow (Gemini 2.5 Flash pricing)

**Response Times**:
- Vision analysis: 3-5 seconds
- Recipe generation: 5-8 seconds
- Total user wait time: ~10-15 seconds

**Rate Limits**: Gemini free tier allows sufficient requests for demo purposes

**Scalability**: [Not Available] - Current architecture is single-user, synchronous. No load testing performed.
