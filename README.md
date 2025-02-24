# **AweraIA Project Development Approach**

---

## **Comparison of the Two Approaches**

### **Approach 1: Using Vector Store with Similarity Search**
#### **Workflow:**
1. Store match data (CSV) in **MongoDB**.
2. Convert match insights and team reports into **embeddings** and store them in **a vector database (e.g., Pinecone, Weaviate, or FAISS)**.
3. When a coach queries for opponent analysis, perform **similarity search** to fetch the most relevant historical insights.
4. Use **GPT-4o** to summarize and generate responses based on retrieved documents.

#### **Pros:**
- **Fast similarity-based search** for contextual retrieval.
- **Efficient for unstructured data** like match reports and scouting notes.
- Works well for **retrieving insights from past opponent reports**.

#### **Cons:**
- Doesn't work well for **real-time data fetching** (e.g., retrieving the latest standings or upcoming matches).
- **Needs frequent embedding updates** to ensure recent data is included.

---

### **Approach 2: Using Function Calling (API-Based Data Retrieval)**
#### **Workflow:**
1. Store structured match data in **MongoDB**.
2. Set up API endpoints to **retrieve specific details** (e.g., team SWOT, standings, goal trends).
3. Use **GPT-4o's function calling** to dynamically fetch **only relevant** data when answering queries.
4. **Real-time API calls** ensure responses are always based on the latest match data.

#### **Pros:**
- **More structured approach**—GPT-4o can fetch specific details via function calls.
- **Ensures up-to-date information** from MongoDB.
- Easier to **scale with new features** in the future.

#### **Cons:**
- Requires **well-designed API architecture**.
- **Higher latency** compared to pre-retrieved vector searches.

---

## **Best Approach: API-Based Function Calling (Approach 2)**

✅ **The API-based function calling approach is the better choice** for this project because:
- **Match data and standings need real-time updates**, which API calls handle well.
- Coaches need specific insights, and function calling ensures **precise, structured data retrieval**.
- It aligns with **custom GPT implementations**, allowing dynamic data fetching.
- **No need to re-embed data frequently**, making it **easier to maintain**.

---

## **Step-by-Step Guide to Implement Approach 2 (API-Based Function Calling)**

### **1️⃣ Data Storage (MongoDB)**
- Store structured data from the CSVs into MongoDB collections:
  - `tournaments`
  - `standings`
  - `teams`
  - `matches`
  - `scouting_reports`
- Use **indexes** on important fields (e.g., team names, match dates) to improve query performance.

#### **Tech Stack for Data Storage**
- **MongoDB** (as the NoSQL database)
- **Mongoose (Node.js)** or **PyMongo (Python)** to interact with MongoDB

---

### **2️⃣ API Development**
- **Set up FastAPI (Python) or Express.js (Node.js)** to serve APIs.
- Create the following API endpoints:

  | Endpoint | Description |
  |----------|------------|
  | `/team/{team_name}/insights` | Returns strengths, weaknesses, goal trends |
  | `/standings/{league}` | Returns current league standings |
  | `/match/{team_name}/history` | Returns past matches of a team |
  | `/swot/{team_name}` | Returns team’s SWOT analysis |

#### **Tech Stack for API**
- **FastAPI (Python) or Express.js (Node.js)**
- **MongoDB Atlas (Cloud Storage)**
- **JWT Authentication** (for securing API access)

---

### **3️⃣ GPT-4o Function Calling Integration**
- Connect GPT-4o with **OpenAI’s function calling API**.
- Define functions that map to API endpoints (OpenAI API Specification):
  
  ```python
  functions = [
      {
          "name": "get_team_insights",
          "description": "Fetch insights for a specific team",
          "parameters": {
              "team_name": "string"
          }
      },
      {
          "name": "get_standings",
          "description": "Fetch current league standings",
          "parameters": {
              "league_name": "string"
          }
      }
  ]
  ```

- When a coach asks:
  - **"What are my next opponent’s strengths?"** → GPT-4o calls `get_team_insights()`.
  - **"Where does my team rank in the league?"** → GPT-4o calls `get_standings()`.

#### **Tech Stack for Function Calling**
- **OpenAI GPT-4o API**
- **Python `openai` library** for function calling
- **FastAPI/Node.js for backend API**

---

### **4️⃣ AI Response Generation**
- Once the API fetches data, GPT-4o **summarizes insights**.
- Example response:

  **Coach Query:** *"How does Lansdowne play?"*
  
  **API fetches:**
  ```json
  {
    "strengths": ["Strong early-game offense", "Quick counter-attacks"],
    "weaknesses": ["Weak on set pieces", "Struggles under high press"],
    "goal_stats": { "scored": 65, "conceded": 26 }
  }
  ```

  **GPT-4o generates:**
  *"Lansdowne is aggressive early in games, scoring 65 goals this season. They struggle on set pieces and under high pressure. Exploit these weaknesses for an edge."*

---

## **Recommended Tools & Libraries**

| **Component** | **Tool/Library** |
|--------------|----------------|
| **Database** | MongoDB Atlas, Mongoose (Node.js) |
| **Backend** | FastAPI (Python) or Express.js (Node.js) |
| **API Hosting** | AWS Lambda, Vercel, DigitalOcean |
| **Vector Store (if needed later)** | Pinecone, Weaviate |
| **Function Calling** | OpenAI GPT-4o API |
| **Auth** | JWT Authentication |

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>



# **Integrating Python Function Calling into Node.js Website**

## **Step-by-Step Guide to Integration**

### **1️⃣ Set Up Express.js API Endpoints in Node.js**

Fro example: Create an **Express.js API** to fetch soccer team insights and expose its endpoint:

```javascript
const express = require("express");
const mongoose = require("mongoose");

const app = express();
app.use(express.json());

const teamsData = {
  "Lansdowne": { strengths: ["Strong attack"], weaknesses: ["Weak defense"] },
  "Manhattan Celtic": { strengths: ["High possession"], weaknesses: ["Slow transitions"] }
};

// Expose the endpoint
app.get("/api/team/:team_name", (req, res) => {
    const teamName = req.params.team_name;
    const data = teamsData[teamName] || { message: "Team not found" };
    res.json(data);
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

---

### **2️⃣ Write Python Function to Call Express.js API**

Use `requests` to fetch data from **Express.js API** and process it with **GPT-4o function calling**.

```python
import openai
import requests

openai.api_key = "your-openai-api-key"

# Define the function to call the Express.js API
def get_team_insights(team_name):
    url = f"http://localhost:5000/api/team/{team_name}"
    response = requests.get(url)
    return response.json()


# Function Details  
functions = [
    {
        "name": "get_team_insights",
        "description": "Fetch insights for a specific soccer team",
        "parameters": {
            "team_name": {"type": "string"}
        }
    }
]

# Calling AI Model (GPT-4o) to generate the response
def generate_response(user_query):
    chat_response = openai.ChatCompletion.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "You are a soccer analytics assistant."},
            {"role": "user", "content": user_query}
        ],
        functions=functions
    )

  # Extract the function call and arguments
    if chat_response["choices"][0]["message"].get("function_call"):
        function_name = chat_response["choices"][0]["message"]["function_call"]["name"]
        function_args = chat_response["choices"][0]["message"]["function_call"]["arguments"]
        if function_name == "get_team_insights":
            return get_team_insights(function_args["team_name"])
    return "Sorry, I couldn't process your request."

user_query = "What are the strengths and weaknesses of Lansdowne?"
print(generate_response(user_query))
```

---

### **3️⃣ Expose Python API Using FastAPI (Optional)**

```python
from fastapi import FastAPI
import requests

app = FastAPI()


```

---

### **4️⃣ Connect Chatbot to Express.js and Python**

Modify **Express.js** chatbot route to call **Python API**.

```javascript
const axios = require("axios");

app.post("/api/chatbot", async (req, res) => {
    const userQuery = req.body.query;

    try {
        const pythonAPIResponse = await axios.get(`http://localhost:8000/api/get_team_insights/Lansdowne`);
        res.json({ response: pythonAPIResponse.data });
    } catch (error) {
        res.status(500).json({ error: "Python API call failed" });
    }
});
```

---

## **Final Flow of Integration**

```
User Query → Chatbot (Node.js) → Python Function Calling → Express.js API → MongoDB → JSON Data → GPT-4o Summarized Response → User
```

SAS


