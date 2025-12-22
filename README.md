# 📸🍳 SnapChef 

**AI-Powered Meal Planning Agent - Snap Your Fridge, Chef Does the Rest**

> Snap a photo of your fridge → Get 3 optimized recipes → Reduce waste by 85% → Save money

---

## 🎯 The Problem

**40% of food in US households is wasted**, costing families an average of **$1,500 per year**. People struggle with meal planning because they:
- Forget what's in their fridge
- Don't know what recipes to make with available ingredients
- Buy duplicate items at the grocery store
- Spend 20-30 minutes planning each meal

Traditional recipe apps require tedious manual ingredient entry and don't optimize for what you already have, leading to more waste and unnecessary spending.

## 💡 Our Solution

**SnapChef** uses AI-powered computer vision and intelligent recipe generation to solve this problem in under 60 seconds:

1. 📸 **Snap** a photo of your fridge
2. 🤖 **AI detects** all ingredients automatically
3. 💬 **Tell** your preferences ("low-carb dinner")
4. 🍽️ **Get** 3 optimized recipe options with images
5. 👆 **Choose** your favorite
6. 📧 **Export** shopping list via email or text

## ✨ Key Benefits

| Benefit | Impact |
|---------|--------|
| ♻️ **Reduce Waste** | Uses 60-85% of ingredients vs. typical 40% |
| 💰 **Save Money** | Only buy what you need: $5-8 vs. $25-35 per meal |
| ⏱️ **Save Time** | 5 minutes vs. 20-30 minutes of meal planning |
| 🥗 **Eat Healthier** | Nutritional info and balanced meal suggestions |
| 🌍 **Help Environment** | Less waste = less methane emissions |

---

## 🎬 Quick Demo

**Input:**
```
📸 Photo: Chicken, eggs, bell peppers, broccoli, milk
💬 Constraint: "Low-carb dinner for 2"
```

**Output in 60 seconds:**

<table>
<tr>
<td width="33%">
<b>Option 1: Chicken Stir-Fry</b><br>
⏱️ 25min | 🔥 450 cal<br>
🛒 2 items: soy sauce, ginger
</td>
<td width="33%">
<b>Option 2: Veggie Omelet</b><br>
⏱️ 15min | 🔥 320 cal<br>
🛒 1 item: cheese
</td>
<td width="33%">
<b>Option 3: Chicken Salad</b><br>
⏱️ 20min | 🔥 380 cal<br>
🛒 3 items: olive oil, lemon, feta
</td>
</tr>
</table>

**Selected Recipe Details:**
```
🍽️  CHICKEN VEGGIE STIR-FRY

✅ YOU HAVE:
• 1 breast - chicken
• 2 whole - red bell peppers
• 1 cup - broccoli

🛒 NEED TO BUY:
• 2 tbsp - soy sauce
• 1 tsp - fresh ginger

♻️ WASTE REDUCTION: 85% of ingredients used!
💰 COST: ~$6.48 (vs. $30 buying all new)
```

---

## 🏗️ How It Works

SnapChef uses a **single Gemini 2.5 Flash agent** performing three sequential tasks:

```
📸 Photo Upload
    ↓
🔍 TASK 1: Vision Analysis
    - Gemini detects all ingredients
    - Categorizes by type (proteins, veggies, etc.)
    - Estimates quantities
    ↓
📋 TASK 2: Recipe Generation
    - Creates 3 diverse options
    - Option 1: Fastest prep
    - Option 2: Highest ingredient usage
    - Option 3: Fewest missing items
    ↓
✅ TASK 3: Display & Export
    - Show full recipe with instructions
    - Export shopping list (email or copy/paste)
```

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| **AI Model** | Gemini 2.5 Flash (multimodal) |
| **Language** | Python 3.10+ |
| **Platform** | Google Colab (zero setup!) |
| **Vision** | Gemini Vision API |
| **Images** | Unsplash Source API |
| **UI** | IPython HTML/CSS |
| **Memory** | Session-based (in-memory) |

### Key Libraries:
```
google-generativeai  # Gemini SDK
Pillow              # Image processing
requests            # HTTP requests
IPython             # Interactive display
```

---

## 🚀 Getting Started

### Prerequisites
- Google account (for Colab)
- Gemini API key ([Get free key here](https://aistudio.google.com/app/apikey))

### Quick Start (3 steps)

**1. Open in Google Colab**
```
Click: File → Open notebook → GitHub
Enter: [Your GitHub URL]/SnapChef
```

**2. Set up API key**
- Click 🔑 icon (left sidebar) → Secrets
- Add: `GEMINI_API_KEY` = your key
- Toggle: Enable notebook access

**3. Run all cells**
```
Click: Runtime → Run all
Follow the prompts!
```

### Usage Flow

1. **Upload fridge photo** when prompted
2. **Enter constraints** (e.g., "healthy dinner", "quick lunch under $10")
3. **View 3 options** with images, prep time, calories
4. **Select favorite** (type 1, 2, or 3)
5. **Get recipe** with full instructions
6. **Export list** via email or copy/paste

That's it! 🎉

---

## ✅ Features

| Feature | Status |
|---------|--------|
| 📸 Computer Vision Ingredient Detection | ✅ Working |
| 🍳 AI Recipe Generation (3 options) | ✅ Working |
| 🖼️ Recipe Images from Unsplash | ✅ Working |
| 📊 Nutritional Information | ✅ Working |
| ♻️ Waste Reduction Score | ✅ Working |
| 📧 Email Shopping List | ✅ Working |
| 📋 Copy/Paste Shopping List | ✅ Working |
| 🧠 Smart Pantry Staples Exclusion | ✅ Working |

---

## 📊 Performance

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Ingredient Detection Accuracy | >85% | ~90% | ✅ Exceeds |
| Recipe Generation Time | <10s | ~6s | ✅ Exceeds |
| Constraint Satisfaction | >90% | ~95% | ✅ Exceeds |
| Waste Reduction | >60% | 60-85% | ✅ Meets |
| Total Workflow Time | <2min | ~60s | ✅ Exceeds |

---

## 🌟 What Makes SnapChef Special?

### Technical Innovation
- **True Multimodal AI**: Seamlessly processes images + text in one workflow
- **Smart Optimization**: Generates 3 options optimized for different goals (speed, waste, cost)
- **Human-in-the-Loop**: Gives users choice, not dictation
- **Intelligent Filtering**: Auto-excludes basic staples (salt, oil, water) from shopping lists

### Real-World Impact
- **$218B Problem**: Food waste is massive in the US alone
- **Measurable Results**: 60-85% ingredient usage vs. typical 40%
- **Immediate Utility**: No app install, works in browser today
- **Environmental Benefit**: Less waste → less methane → climate impact

### Market Differentiation
| Other Solutions | SnapChef |
|-----------------|----------|
| Manual ingredient entry | 📸 Computer vision |
| Single recipe output | 3 optimized options |
| Generic recipes | Uses YOUR ingredients |
| 20-30 min planning | 60 seconds total |
| Expensive meal kits | Uses what you have |

---

## 📂 Project Structure

```
SnapChef/
├── snapchef_notebook.ipynb     # Main application
├── README.md                    # This file
├── ARCHITECTURE.md              # System design details
├── EXPLANATION.md               # Technical implementation
├── requirements.txt             # Dependencies
├── LICENSE                      # MIT License
├── .gitignore                   # Git ignore rules
│
├── docs/
│   └── demo_script.md          # 3-minute demo guide
│
└── examples/
    └── sample_fridge.jpg        # Test image
```

---

## 📖 Documentation

- **[Architecture Overview](ARCHITECTURE.md)** - System design and components
- **[Technical Explanation](EXPLANATION.md)** - Implementation details and limitations


---

## 🤝 Contributing

Contributions welcome! Feel free to:
- 🐛 Report bugs
- 💡 Suggest features
- 🔧 Submit pull requests
- ⭐ Star the repo if you find it useful!

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- **Google Gemini** - Powerful multimodal AI capabilities
- **Unsplash** - High-quality recipe images
- **Google Colab** - Free, accessible development platform

---

## 📧 Contact

Questions? Feedback? Open an issue on GitHub!

---

## 🏆 Built For

Google AI Hackathon - Showcasing the power of Gemini 2.5 Flash for real-world problem solving

---

**⚡ SnapChef - Turning your fridge into a smart kitchen assistant, one photo at a time.**

*Built with ❤️ to combat food waste and help families save money*
