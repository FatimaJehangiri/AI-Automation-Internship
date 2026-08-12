# Day 13 — Generative AI & LLM Fundamentals

## 📌 Objective
This task is designed to build a foundational understanding of Large Language Models (LLMs) and their practical applications. By the end of this module, I will have:
- Understood the key concepts of Generative AI, tokens, context windows, and hallucinations.
- Set up free/low-cost access to two leading LLM providers: **Google Gemini** and **Groq**.
- Successfully tested both APIs using **Postman**.
- Developed a reasoned comparison of which provider is best suited for **classification** vs. **long-context** workloads.

- ## ⚙️ Prerequisites
Before starting, ensure you have the following:
- A **Google account** (for Gemini API).
- An **email/Google account** (for Groq Console).
- **Postman** installed (or the Postman web app).
- *(Optional but recommended)* **Python 3.8+** and `pip` installed, along with a code editor.

---

## 🔑 Step 1: Get Your API Keys

### 1.1 Google Gemini API Key (via Google AI Studio)
1. Navigate to **[Google AI Studio](https://aistudio.google.com/)**.
2. Sign in with your Google account.
3. In the left sidebar, click **"Get API key"**.
4. Click **"Create API key"**.
5. Choose an existing Google Cloud project or create a new one.
6. **Copy the generated API key** and save it securely (you will need it in Postman).

### 1.2 Groq API Key (via Groq Console)
1. Navigate to the **[Groq Console](https://console.groq.com/)**.
2. Sign up or log in (you can use Google OAuth for simplicity).
3. In the left sidebar, expand the **"API Keys"** section.
4. Click **"Create API Key"**.
5. Give it a descriptive name, e.g., `Postman-Task-Day13`.
6. **Copy the key immediately** (Groq does not show it again after creation).

---

## 🧪 Step 2: Testing APIs with Postman

### 2.1 Import the Collections
1. Open Postman.
2. Click the **"Import"** button (top left).
3. Select the **"Upload Files"** tab and drag the two JSON files located in the `/postman` folder of this repository.
4. Both collections (`Gemini API Test` and `Groq API Test`) will appear in your Collections panel.

### 2.2 Set Up Environment Variables (Highly Recommended)
To avoid pasting keys into every request, set up a Postman Environment:
1. Click the **"Environments"** cogwheel (top right).
2. Click **"Add"**.
3. Name it `Day 13 - AI`.
4. Add two variables:
   - `GEMINI_API_KEY` (Type: `Secret`)
   - `GROQ_API_KEY` (Type: `Secret`)
5. Paste your actual API keys into the **"Current Value"** field.
6. Click **"Save"** and select this environment from the dropdown in the top right corner.

### 2.3 Test Google Gemini
**Steps:**
1. Open the `Gemini API Test` collection and select the request **"Gemini - Generate Content"**.
2. Ensure your environment is selected so `{{GEMINI_API_KEY}}` resolves correctly.
3. Click **"Send"**.

### Test Groq
Open the Groq API Test collection and select "Groq - Chat Completion".
Ensure your environment is selected so {{GROQ_API_KEY}} resolves correctly.
Click "Send".

### Troubleshooting: Common Errors & Fixes
## Error Messages
1.Header name must be a valid HTTP token
2.404: models/gemini-1.5-flash is not found
3.Invalid JSON payload
4.API key not valid

## Cause
1.Using gemini api key with a space in the header name.
2.The specific model is unavailable or deprecated in your region/project.
3.Syntax error in the request body (e.g., single quotes, trailing commas).
4.You are passing the key in the wrong place or using an incorrect header.

## Fix
1.Use x-goog-api-key as the header name, or append ?key=YOUR_KEY to the URL.
2.1. Send a GET request to https://generativelanguage.googleapis.com/v1beta/models?key=YOUR_KEY.
  2. Pick an active model (e.g., gemini-1.5-pro or gemini-1.0-pro).
  3. Update your URL to use this exact name.
3.Use exactly the JSON format provided in the Postman collections. Ensure double quotes " are used for keys and strings.
4.For Gemini: Pass ?key=YOUR_KEY OR use header x-goog-api-key.
For Groq: Use header Authorization: Bearer YOUR_KEY.

This task provided hands-on experience with the practical setup of LLM APIs. Understanding the trade-offs between speed/cost (Groq) and context capacity/retention (Gemini) is critical for designing efficient AI-powered systems.
