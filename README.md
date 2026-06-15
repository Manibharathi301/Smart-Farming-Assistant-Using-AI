# 🌾 Uzhavan — AI-Powered Agricultural Assistant for Indian Farmers

> **Uzhavan** (உழவன்) means *farmer* in Tamil. This platform empowers Indian farmers with AI-driven soil analysis, crop recommendations, irrigation guidance, and government scheme discovery — in their native language.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Setup & Installation](#setup--installation)
- [Environment Variables](#environment-variables)
- [API Endpoints](#api-endpoints)
- [ML Models](#ml-models)
- [Supported Languages](#supported-languages)
- [Screenshots](#screenshots)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

Uzhavan is a full-stack Flask web application that uses machine learning and generative AI to assist Indian farmers with:

- Identifying soil types from photos
- Getting crop recommendations based on soil and climate data
- Receiving irrigation guidance
- Discovering government agricultural schemes by state
- Chatting with an AI assistant in regional languages
- Listening to responses via text-to-speech

The platform is designed to be accessible to farmers across India with multilingual support in Tamil, Telugu, Kannada, Malayalam, Hindi, and English.

---

## Features

| Feature | Description |
|---|---|
| 🪨 Soil Detection | Upload a soil image → EfficientNet-B0 classifies it as Alluvial, Black, Clay, or Red soil |
| 🌱 Crop Recommendation | Input NPK, temperature, humidity, pH, rainfall → Get top 5 suitable crops with probabilities |
| 💧 Irrigation Guidance | ML model predicts irrigation need level (Very Low → Very High) |
| 📈 Yield Estimation | Heuristic estimate of crop yield in tons/hectare |
| 🐛 Pest Detection | Soil-based heuristic pest risk assessment |
| 🏛️ Government Schemes | State-wise agricultural scheme lookup with eligibility info |
| 🤖 AI Chatbot | Gemini-powered conversational assistant for farming queries |
| 🔊 Text-to-Speech | Convert any response to audio in the user's language |
| 🌐 Multilingual | Full translate pipeline (input → English → AI → target language) |
| 🔐 User Auth | Register/login with phone number or email + bcrypt-hashed passwords |

---

## Tech Stack

**Backend**
- Python 3.10+
- Flask + Flask-CORS
- PyTorch (EfficientNet-B0 for soil classification)
- scikit-learn (crop & irrigation models via joblib)
- Google Gemini API (`gemini-2.5-flash`) for chat & translation
- gTTS (Google Text-to-Speech)
- MongoDB (user auth + translation caching)
- python-dotenv

**Frontend**
- HTML/CSS/JS (served via Flask templates & static files)

**Infrastructure**
- Deployable on any server with Python 3.10+
- Port configurable via `PORT` environment variable (default: 1000)

---

## Project Structure

```
uzhavan/
├── app.py                    # Main Flask application
├── .env                      # Environment variables (not committed)
├── .env.example              # Template for environment variables
├── requirements.txt          # Python dependencies
├── README.md
│
├── model/                    # Trained ML model files
│   ├── efficientnet_soil.pth       # Soil classification model
│   ├── crop_model.pkl              # Crop recommendation model
│   └── irrigation_model.pkl        # Irrigation prediction model
│
├── data/
│   └── schemes.py            # State-wise government scheme data
│
├── templates/                # Jinja2 HTML templates
│   ├── login.html
│   └── index.html
│
├── static/                   # Static assets
│   ├── chatindex.html
│   ├── translations.json
│   └── ...
│
├── videos/                   # Optional crop cultivation videos
├── audio_output/             # Auto-generated TTS audio files
└── logs/
```

---

## Setup & Installation

### Prerequisites

- Python 3.10+
- MongoDB instance (local or Atlas)
- Google Gemini API key
- Model `.pth` and `.pkl` files in `model/`

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/uzhavan.git
cd uzhavan
```

### 2. Create a Virtual Environment

```bash
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure Environment Variables

```bash
cp .env.example .env
# Edit .env with your credentials
```

### 5. Add Model Files

Place your trained model files in the `model/` directory:
- `efficientnet_soil (1).pth`
- `crop_model (2) (1).pkl`
- `irrigation_model (1).pkl`

### 6. Run the Application

```bash
python app.py
```

The app will be available at `http://localhost:1000`

---

## Environment Variables

Create a `.env` file in the project root based on the template below:

```env
# MongoDB connection string
MONGODB_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/?retryWrites=true&w=majority

# Google Gemini API key
GEMINI_API_KEY=your_gemini_api_key_here

# Flask secret key (auto-generated if not set)
SECRET_KEY=your_random_secret_key_here

# Server port (optional, default: 1000)
PORT=1000
```

> ⚠️ **Never commit your `.env` file.** Add it to `.gitignore`.

---

## API Endpoints

### Authentication

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/register` | Register a new user |
| `POST` | `/login` | Login with phone/email + password |

### AI / ML Features

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/predict_soil` | Upload soil image → soil type + pest info |
| `POST` | `/recommend_crop` | Input farm data → crop + irrigation + yield |
| `POST` | `/government_aids` | State + land size → applicable schemes |
| `POST` | `/api/grok` | Send message to AI chatbot |
| `POST` | `/api/tts` | Convert text to speech audio |

### Utilities

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | System health check |
| `GET` | `/model_info` | Crop model metadata |
| `GET` | `/audio/<filename>` | Serve TTS audio file |
| `GET` | `/videos/<filename>` | Serve crop cultivation video |

### Example: Crop Recommendation Request

```json
POST /recommend_crop
{
  "nitrogen": 90,
  "phosphorus": 42,
  "potassium": 43,
  "temperature": 25,
  "humidity": 82,
  "ph": 6.5,
  "rainfall": 200,
  "soil_type": "Alluvial",
  "lang": "ta"
}
```

### Example: Chat Request

```json
POST /api/grok
{
  "message": "நெல் சாகுபடிக்கு எந்த உரம் நல்லது?",
  "language": "ta"
}
```

---

## ML Models

### Soil Classification (`EfficientNet-B0`)
- **Input:** RGB image (224×224)
- **Output:** One of `Alluvial Soil`, `Black Soil`, `Clay Soil`, `Red Soil`
- **Fallback:** None (model required)

### Crop Recommendation
- **Input:** N, P, K, temperature, humidity, pH, rainfall, soil type
- **Output:** Top 5 crops with probability scores
- **Fallback:** Heuristic rule-based system if model fails

### Irrigation Prediction
- **Input:** Same features + soil type encoding
- **Output:** `Very low` / `Low` / `Moderate` / `High` / `Very high`
- **Fallback:** Heuristic system based on crop water needs and rainfall

---

## Supported Languages

| Code | Language |
|---|---|
| `en` | English |
| `ta` | Tamil |
| `te` | Telugu |
| `kn` | Kannada |
| `ml` | Malayalam |
| `hi` | Hindi |

All translations are powered by Google Gemini and cached in MongoDB to reduce API calls.

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m "Add: your feature description"`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a Pull Request

### Code Style
- Follow PEP 8 for Python code
- Add logging for all new routes and model calls
- Include error handling with meaningful error codes

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## Acknowledgements

- [Google Gemini](https://deepmind.google/technologies/gemini/) for AI chat and translation
- [PyTorch](https://pytorch.org/) and [EfficientNet](https://arxiv.org/abs/1905.11946) for image classification
- [gTTS](https://gtts.readthedocs.io/) for multilingual text-to-speech
- [MongoDB Atlas](https://www.mongodb.com/atlas) for cloud database
- Indian Government agricultural data for scheme information

---

*Built with ❤️ to support the backbone of India — our farmers.*
