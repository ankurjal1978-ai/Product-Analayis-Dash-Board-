# Product Analysis Dashboard - Complete Project Guide

## 📋 Project Overview

This is a **Streamlit-based web application** that analyzes products and generates detailed business analysis reports using OpenAI's GPT model through OpenRouter. The app takes a product name as input and uses AI to generate comprehensive reports covering market demand, marketing strategies, technology feasibility, business models, and launch timelines.

---

## 🏗️ Project Structure

```
Product/
├── app.py                 # Main application file (Streamlit UI + AI logic)
├── requirments.txt        # Python package dependencies (note: has typo - should be "requirements.txt")
├── .env                   # Environment variables file (contains API keys) - DO NOT share publicly!
└── PROJECT_GUIDE.md       # This documentation file
```

---

## 📦 Module Breakdown

### 1. **app.py** (Main Application File)

This is the **heart of the project**. It contains:

- **Imports & Setup**: Loads necessary libraries
- **API Configuration**: Connects to OpenRouter (AI service)
- **analyze_product()**: Function that generates AI analysis
- **main()**: Function that creates the user interface
- **Streamlit UI**: Web interface for users to interact with

### 2. **requirments.txt** (Dependencies File)

Lists all Python packages needed to run the project:
- `openai` - OpenAI API client for ChatGPT
- `streamlit` - Web framework for creating the UI
- `python-dotenv` - Loads environment variables from .env file
- `pandas` - Data manipulation library (optional for this project)

### 3. **.env** (Environment Variables)

Stores sensitive information:
```
OPENROUTER_API_KEY=sk-or-v1-xxxxx...
```

**⚠️ SECURITY NOTE**: Never share or commit this file to GitHub!

---

## 🔍 Detailed Code Explanation - Line by Line

### **IMPORTS SECTION** (Lines 1-15)

```python
import os
```
- **Purpose**: Access operating system functions (like reading file paths)
- **Used for**: Getting the `.env` file location

```python
import streamlit as st
```
- **Purpose**: Import Streamlit framework
- **Used for**: Creating the web interface (`st.title()`, `st.button()`, etc.)
- **Think of it as**: A template builder that creates professional websites without HTML/CSS knowledge

```python
from datetime import datetime
```
- **Purpose**: Import date/time tools
- **Used for**: Getting current date/time (`datetime.now()`)
- **Why we need it**: The AI prompt includes the current month/year for context

```python
from openai import OpenAI
```
- **Purpose**: Import OpenAI client library
- **Used for**: Connecting to AI services and sending requests
- **Note**: Works with OpenRouter, not just OpenAI directly

```python
from dotenv import load_dotenv
```
- **Purpose**: Import tool to read `.env` files
- **Used for**: Loading API keys from the `.env` file securely
- **Why needed**: We can't hardcode API keys in the code (security risk!)

```python
load_dotenv(dotenv_path=os.path.join(os.path.dirname(__file__), '.env'))
```
- **What it does**: 
  - `os.path.dirname(__file__)` = Get the current file's directory
  - `os.path.join()` = Combine path with '.env' filename
  - `load_dotenv()` = Load variables from that `.env` file
- **Result**: Now `os.getenv("OPENROUTER_API_KEY")` will work

---

### **API SETUP SECTION** (Lines 17-22)

```python
client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv("OPENROUTER_API_KEY"),
)
```

- **What it does**: Creates a connection to OpenRouter API
- **Parameters explained**:
  - `base_url="https://openrouter.ai/api/v1"` = The server address to send requests to
  - `api_key=os.getenv("OPENROUTER_API_KEY")` = Retrieves API key from .env file
- **Think of it as**: Setting up a phone connection with an AI service (OpenRouter)
- **Why OpenRouter?**: It lets us access 300+ AI models through one API

```python
MODEL = "openai/gpt-4.1-mini"
```
- **What it does**: Specifies which AI model to use
- **Format**: `provider/model-name`
- **Why mini?**: It's faster and cheaper than full GPT-4, but still very capable
- **Think of it as**: Choosing which version of ChatGPT to use

---

### **analyze_product() FUNCTION** (Lines 27-91)

#### **Function Definition**
```python
def analyze_product(product_name):
    """Single agent: one OpenAI call that returns a full product-analysis report."""
```
- **What it does**: Takes a product name and returns an AI-generated analysis
- **Input**: `product_name` (string) - e.g., "Wireless Charging Phone"
- **Output**: Full product analysis report (string)

#### **Step 1: Get Current Date**
```python
current_date = datetime.now().strftime("%b %Y")
```
- **What it does**: Gets today's date and formats it
- **Example output**: "Aug 2026"
- **Why we need it**: The AI should know the current time context

#### **Step 2: Create System Prompt**
```python
system_prompt = (
    "You are a senior product and business analyst. You write clear, practical, "
    "well-structured product analysis reports for founders and business teams."
)
```
- **What it does**: Tells the AI what role to play
- **Think of it as**: Giving ChatGPT a job description
- **Result**: AI will respond like a professional business analyst, not a casual chatbot

#### **Step 3: Create User Prompt**
```python
user_prompt = f"""
Write a detailed product analysis report for: {product_name}.
Current month is {current_date}.
Cover the following in one flowing, well-organized report (use markdown headings and bullet points):
- Market demand and the ideal customer profile
- Marketing strategies to reach the widest possible audience (at least 5 points)
- Technology and manufacturing feasibility / key requirements (at least 5 points)
- Business model: scalability and revenue streams (at least 5 points)
- A concise Business Plan, Goals, and a launch Timeline
Keep it insightful and actionable.
"""
```
- **What it does**: Asks the AI to analyze the product with specific requirements
- **The `f"..."` part**: Means we can insert variables (product_name, current_date) into the string
- **Example of filled prompt**:
  ```
  Write a detailed product analysis report for: Smart Water Bottle.
  Current month is Aug 2026.
  Cover the following...
  ```

#### **Step 4: Send Request to AI**
```python
response = client.chat.completions.create(
    model=MODEL,
    extra_headers={
        "HTTP-Referer": "",
        "X-OpenRouter-Title": "Product Analysis Dashboard",
    },
    extra_body={},
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt},
    ],
)
```
- **What it does**: Sends the conversation to OpenRouter/ChatGPT and waits for response
- **Parameters**:
  - `model=MODEL` = Use GPT-4-mini model we defined earlier
  - `extra_headers` = Metadata about our app (for logging purposes)
  - `messages` = A conversation history (like a chat)
    - First message: Job description (system prompt)
    - Second message: What we want analyzed (user prompt)
- **Think of it as**: Having a conversation with ChatGPT

#### **Step 5: Return the Analysis**
```python
return response.choices[0].message.content
```
- **What it does**: Extracts the AI's response and returns it
- **Breakdown**:
  - `response` = The full response object from the API
  - `.choices[0]` = Take the first response (we only asked for 1)
  - `.message.content` = Get the actual text of the message
- **Think of it as**: Reading ChatGPT's answer and passing it to the caller

---

### **main() FUNCTION - UI SECTION** (Lines 94-160)

#### **Page Title**
```python
st.title("Product Analysis Dashboard")
```
- **What it does**: Displays a large title at the top of the webpage
- **Looks like**: # Product Analysis Dashboard (in markdown)

#### **Custom Styling**
```python
st.markdown(
    """
    <style>
    .reportview-container { max-width: 1200px; padding-top: 2rem; }
    h3 { color: #1f77b4; margin-top: 1rem; }
    .stExpander { border: 1px solid #f0f2f6; border-radius: 4px; margin-bottom: 1rem; }
    .stMarkdown { line-height: 1.6; }
    </style>
    """,
    unsafe_allow_html=True,
)
```
- **What it does**: Adds custom CSS styling to make the UI look professional
- **Breakdown**:
  - `max-width: 1200px` = Don't make content wider than 1200 pixels
  - `color: #1f77b4` = Make headings blue
  - `border-radius: 4px` = Slightly rounded corners on boxes
  - `line-height: 1.6` = Add space between text lines for readability

#### **Input Text Box**
```python
product_name = st.text_input("Enter the product name you want to analyze:", "")
```
- **What it does**: Creates a text input field where users type the product name
- **Parameters**:
  - First parameter = Label/question shown to user
  - Second parameter = Default value (empty string "")
- **Returns**: What the user typed (stored in `product_name` variable)

#### **Analyze Button**
```python
if st.button("Analyze Product"):
```
- **What it does**: Creates a clickable button
- **What happens**: When clicked, the code block inside executes

#### **Validation - Check if Product Name Entered**
```python
if not product_name:
    st.error("Please enter a product name before starting the analysis.")
else:
```
- **What it does**: Checks if user actually typed something
- **If empty**: Shows error message
- **If filled**: Continues to analysis

#### **Loading Placeholder**
```python
loading_placeholder = st.empty()
loading_placeholder.info(f"Starting analysis for '{product_name}'... Please wait.")
```
- **What it does**: 
  - Creates an empty space in the UI
  - Shows "Starting analysis..." message
- **Why needed**: Gives user feedback that something is happening

#### **Try-Except Error Handling**
```python
try:
    with st.spinner("Analyzing product... This may take a few moments."):
        report = analyze_product(product_name)
    
    loading_placeholder.empty()
    st.subheader("Analysis Results")
    
    with st.expander(f"Report: {product_name}", expanded=True):
        st.markdown(report)
        
except Exception as e:
    loading_placeholder.empty()
    st.error(f"An error occurred: {str(e)}")
```
- **What it does**: Safely runs the analysis and catches any errors
- **Breakdown**:
  - `try:` = Try to do this code
  - `with st.spinner(...)` = Show loading spinner while waiting
  - `report = analyze_product(product_name)` = Call the AI analysis function
  - `loading_placeholder.empty()` = Clear the "Starting..." message
  - `st.subheader()` = Show "Analysis Results" heading
  - `st.expander()` = Create an expandable box (like an accordion) with the report
  - `except Exception as e:` = If anything goes wrong, catch the error
  - `st.error()` = Show the error message to user

#### **Main Entry Point**
```python
if __name__ == "__main__":
    main()
```
- **What it does**: Checks if this file is being run directly (not imported)
- **If true**: Runs the `main()` function
- **Why needed**: Allows this file to be both run directly AND imported by other files

---

## ⚠️ Errors We Encountered & How We Fixed Them

### **ERROR 1: Missing API Key Credentials**

**Error Message:**
```
openai.OpenAIError: Missing credentials. Please pass an `api_key`, `workload_identity`, 
`admin_api_key`, or set the `OPENAI_API_KEY` or `OPENAI_ADMIN_KEY` environment variable.
```

**What was wrong:**
- The `.env` file had `OPEN_ROUTER_API_KEY` (with underscores)
- But the code was looking for `OPENROUTER_API_KEY` (without underscores)
- The names didn't match!

**How we fixed it:**
Changed in `.env` file:
```
❌ OPEN_ROUTER_API_KEY="sk-or-v1-..."
✅ OPENROUTER_API_KEY=sk-or-v1-...
```

### **ERROR 2: API Key Had Extra Quotes**

**What was wrong:**
```
OPENROUTER_API_KEY="sk-or-v1-..."  ❌ (quotes are read as part of the key)
```

**Why it's a problem:**
- The quotes were included in the actual key value
- API would receive: `"sk-or-v1-..."` (with quotes)
- Instead of: `sk-or-v1-...` (without quotes)

**How we fixed it:**
```
OPENROUTER_API_KEY=sk-or-v1-...  ✅ (no quotes needed)
```

### **ERROR 3: .env File Not Loading**

**What was wrong:**
```python
load_dotenv()  # Searches in current working directory
```

**Why it's a problem:**
- Sometimes Python doesn't run from the same directory as `.env`
- Especially when running Streamlit, the working directory might be different

**How we fixed it:**
```python
load_dotenv(dotenv_path=os.path.join(os.path.dirname(__file__), '.env'))
```
- **Explicit path**: Now we tell it exactly where `.env` is located
- `os.path.dirname(__file__)` = Directory of app.py file
- `os.path.join(..., '.env')` = Combine with '.env' filename

---

## 🚀 How to Run the Project

### **Step 1: Install Dependencies**
```bash
pip install -r requirments.txt
```
(Note: Fix the typo in filename - should be `requirements.txt`)

### **Step 2: Create .env File**
Create a file named `.env` in the same folder as `app.py`:
```
OPENROUTER_API_KEY=your_api_key_here
```

### **Step 3: Run the App**
```bash
streamlit run app.py
```

### **Step 4: Access the App**
- Open browser to: `http://localhost:8501`
- Enter a product name (e.g., "AI Fitness Tracker", "Smart Water Bottle")
- Click "Analyze Product"
- Wait for AI to generate the report

---

## 📊 Flow Diagram

```
User Enters Product Name
           ↓
User Clicks "Analyze Product" Button
           ↓
Check if Product Name is Empty
    ├─ Yes → Show Error
    └─ No → Continue
           ↓
Show Loading Spinner
           ↓
Call analyze_product() Function
           ↓
Create System Prompt (job description for AI)
           ↓
Create User Prompt (analysis requirements)
           ↓
Send Both Prompts to OpenRouter API
           ↓
ChatGPT Generates Analysis Report
           ↓
Receive Response from API
           ↓
Clear Loading Spinner
           ↓
Display Report in Expandable Box
           ↓
If Error Occurred → Show Error Message
```

---

## 🎯 Key Concepts for Learners

### **1. Prompting**
- **What**: Instructions we give to the AI
- **Types**: System prompt (role) + User prompt (task)
- **Importance**: Good prompts = better AI responses

### **2. API Keys**
- **What**: Secret credentials to access AI services
- **Where to store**: `.env` file (never in code!)
- **Why secure**: Anyone with your key can use your account

### **3. Error Handling**
- **try-except blocks**: Catch errors gracefully instead of crashing
- **User feedback**: Always tell users what went wrong

### **4. Streamlit**
- **Purpose**: Quickly build data apps without HTML/CSS/JavaScript
- **How it works**: Python functions → Web components automatically
- **Benefit**: Developers focus on logic, not UI design

### **5. Environment Variables**
- **What**: Settings stored outside the code
- **Why**: Keep sensitive data secure and configs flexible
- **How loaded**: `.env` file → Python dictionary via `os.getenv()`

---

## 📝 Common Modifications & Challenges

### **Challenge 1: Different AI Models**
To use a different model, change:
```python
MODEL = "openai/gpt-4.1-mini"  # Try "openai/gpt-4" or "anthropic/claude-3-opus"
```

### **Challenge 2: Add More Analysis Sections**
Modify `user_prompt` to ask for more sections:
```python
user_prompt = f"""
... existing requirements ...
- Competitive analysis (at least 3 points)
- Risk assessment and mitigation strategies
- Estimated budget and ROI projections
"""
```

### **Challenge 3: Save Reports to File**
Add after getting the report:
```python
with open(f"{product_name}_analysis.txt", "w") as f:
    f.write(report)
st.success(f"Report saved to {product_name}_analysis.txt")
```

### **Challenge 4: Add Chat History**
Use Streamlit session state to keep chat history:
```python
if "messages" not in st.session_state:
    st.session_state.messages = []
st.session_state.messages.append({"product": product_name, "report": report})
```

---

## ✅ Testing Checklist

- [ ] App starts without errors
- [ ] Text input accepts product names
- [ ] "Analyze Product" button works
- [ ] Error message shows when no product name entered
- [ ] Loading spinner appears during analysis
- [ ] Report displays correctly
- [ ] Report is readable and well-formatted
- [ ] Can analyze multiple products in one session

---

## 🔗 Resources

- **Streamlit Docs**: https://docs.streamlit.io
- **OpenAI API**: https://platform.openai.com/docs
- **OpenRouter**: https://openrouter.ai
- **Python dotenv**: https://github.com/theskumar/python-dotenv

---

## 📌 Summary

This project demonstrates:
1. **Web App Development** (Streamlit)
2. **AI Integration** (OpenAI/OpenRouter API)
3. **Environment Management** (.env files)
4. **Error Handling** (try-except blocks)
5. **Prompt Engineering** (creating effective AI instructions)
6. **Python Fundamentals** (functions, strings, variables)

All these concepts are essential for modern Python developers! 🚀
