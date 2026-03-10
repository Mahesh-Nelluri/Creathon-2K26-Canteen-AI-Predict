# Creathon-2K26-Canteen-AI-Predict
🍛 CanteenAI — Smart Canteen Demand Predictor

AI-powered food demand forecasting for canteens, mess halls, and institutional kitchens.
Predict portions. Reduce waste. Plan smarter.

Show Image
Show Image
Show Image
Show Image
Show Image
Show Image

📋 Table of Contents

Overview
Live Demo
Features
Tech Stack
Project Structure
Firebase Setup
ML Prediction Engine
Feature Set & Model Design
Design System
Page Sections
Getting Started
Deployment
Firestore Data Schema
Analytics Events
Performance & Accessibility
Roadmap
Contributing
License


🧠 Overview
CanteenAI is a complete, production-grade, single-file web application that helps canteen managers forecast food demand using an AI/ML simulation engine. It addresses a critical problem in institutional catering: up to 40% of food is wasted daily due to poor demand planning.
The app simulates a Weighted Multi-Factor Regression model (inspired by XGBoost Regressor architecture) that takes inputs like day of week, meal time, weather, and special events — and outputs per-food-item portion predictions with confidence scores.
All predictions are persisted to Firebase Firestore and tracked via Firebase Analytics.
Key Metrics (simulated targets)
MetricValueWaste Reduction32%Forecast Accuracy94%Meals Optimized1,200+Model MAE Target< 10 portionsMAPE Target< 10%R² Target> 0.85

🌐 Live Demo

Deploy this file to any static host and open in a browser. No build step required.

open index.html
Or deploy instantly via GitHub Pages, Netlify, or Vercel (see Deployment).

✨ Features
Core Functionality

Interactive Demand Predictor — select day, meal time, weather, and event type to generate real-time portion predictions
5 Food Item Forecasts — Dal Rice, Biryani, Sandwich, Samosa, Chai — each with individual weights and demand curves
Confidence Scoring — multi-factor alignment scoring (72–99%) per prediction
Animated Count-Up Numbers — smooth cubic-ease number animation on result render
Animated CSS Bar Charts — color-coded (🟢 normal / 🟡 high / 🔴 critical) demand visualization
7-Day Forecast Table — full weekly demand overview with trend arrows (↑ ↓ →) and event flags
Prediction History Panel — last 5 predictions fetched live from Firestore

Firebase Integration

Firestore — saves every prediction with full input/output payload + server timestamp
Analytics — logs page_view and prediction_run events with metadata
Live Connection Status — real-time status indicator (🟡 Connecting → 🟢 Firebase Live → 🔴 Offline)
Toast Notifications — confirms save success or failure in real time

UI/UX

Dark / Light Mode — system preference detection + manual toggle + localStorage persistence (no flash on reload)
Sticky Nav — backdrop-filter blur, border-bottom on scroll
Staggered Animations — CSS-only fade-up / scale-in entrance animations with nth-child stagger
Feature Importance Chart — IntersectionObserver-triggered animated horizontal bars
Algorithm Explainer — prose article, pseudocode block (JetBrains Mono), blockquote callout
Token Weight Reference Table — impact pills (High / Medium / Low) for all 8 model weight tokens
Responsive Layout — mobile-first, collapses gracefully to single column
prefers-reduced-motion — disables all animations for users who prefer it


🛠 Tech Stack
LayerTechnologyMarkupSemantic HTML5StylingCSS3 — @layer architecture, CSS custom properties, clamp() fluid typeLogicVanilla JavaScript (ES2020, no frameworks)FontsGoogle Fonts — Syne, Lora, JetBrains MonoDatabaseFirebase Firestore (compat SDK v10)AnalyticsFirebase Analytics (Google Analytics 4)HostingAny static host (GitHub Pages, Netlify, Vercel, Firebase Hosting)BuildNone — zero build step, zero dependencies, single file

📁 Project Structure
canteenai/
│
├── index.html          ← Entire application (HTML + CSS + JS, single file)
├── README.md           ← This file
└── .gitignore          ← (optional) ignore DS_Store, etc.

The entire app lives in one HTML file. No node_modules, no bundler, no framework.


🔥 Firebase Setup
1. Firebase Project
This app is connected to the Firebase project canteen-536c7.
Config KeyValueProject IDcanteen-536c7Auth Domaincanteen-536c7.firebaseapp.comStorage Bucketcanteen-536c7.firebasestorage.appMeasurement IDG-WZYMKBWK1P
2. Enable Firestore

Go to Firebase Console
Click Create Database
Choose Start in test mode (for development)
Select your preferred region

3. Firestore Security Rules
Development (open access):
jsrules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
Production (recommended — add Firebase Auth first):
jsrules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /predictions/{docId} {
      allow read, write: if request.auth != null;
    }
  }
}
4. Enable Analytics

Go to Firebase Console → Analytics
Enable Google Analytics for the project
Analytics events will begin tracking automatically on page load

5. Create Firestore Index (for ordered history query)
The history panel queries:
collection: predictions
orderBy: timestamp DESC
limit: 5
Firebase will prompt you to create a composite index the first time this query runs — click the link in the browser console error, or create it manually in Firebase Console → Firestore → Indexes.

🤖 ML Prediction Engine
The JavaScript simulation engine models a Weighted Multi-Factor Regression system. It runs entirely in the browser — no backend required.
Base Demand (portions per meal)
MealBase PortionsBreakfast80Lunch200Dinner120
Multipliers
Day of Week:
DayMultiplierMonday×1.10Tuesday×1.15Wednesday×1.05Thursday×1.20Friday×1.10Saturday×0.70Sunday×0.60
Weather:
ConditionMultiplierReasonSunny×1.00BaselineRainy×1.15Comfort food spikeCold×1.20Warm food demand increaseHot×0.90Reduced appetite
Special Events:
EventMultiplierReasonNone×1.00BaselineExam Week×1.40Higher campus occupancy, stress eatingFestival×1.60Peak demand, special itemsHoliday×0.50Low attendance
Noise: ±5–8% random variance (uniform(0.92, 1.08)) to simulate real-world demand fluctuation.
Safety Buffer: +5% applied to all outputs before display.
Confidence Scoring Algorithm
jsfunction getConfidence(day, meal, weather, event) {
  let score = 50;                              // base
  if ([1, 3].includes(day)) score += 15;      // Tue/Thu peak days
  if (meal === 'lunch')      score += 10;      // lunch most predictable
  if (weather === 'rainy' || weather === 'cold') score += 8;
  if (event === 'exam' || event === 'festival')  score += 12;
  if (day >= 5)              score -= 10;      // weekends less predictable
  return clamp(score, 72, 99);
}
Status Classification
Ratio (portions / base)StatusColor> 1.35× baseCritical🔴 Red1.10× – 1.35× baseHigh🟡 Amber≤ 1.10× baseNormal🟢 Green

📊 Feature Set & Model Design
The prediction model is designed around 8 feature sets with 40+ variables:
Feature SetKey VariablesImportance1 — Timeday_of_week, meal_time, is_weekend, academic_week~20%2 — Weatherweather_condition, temperature_celsius, humidity_percent~10%3 — Eventsis_exam_week, is_festival, event_severity, event_type~18%4 — Attendancetotal_occupancy, occupancy_rate, attendance_vs_avg~15%5 — Food Itemfood_item_id, item_popularity_rank, is_vegetarian~7%6 — Historicalrolling_avg_7day, rolling_avg_30day, demand_trend~25%7 — Operationalmenu_variety_count, staff_on_duty, prep_capacity_max~3%8 — Feedbackavg_rating_last_7days, repeat_order_rate~2%
Interaction Features (engineered):

day_x_meal — day of week × meal time
event_x_occupancy — event severity × occupancy rate
weather_x_meal — weather condition × meal time
weekend_x_item — is_weekend × food_item_id
exam_x_attendance — is_exam_week × occupancy_rate

Recommended Production Models:
ModelRoleNotesXGBoost RegressorPrimaryBest for tabular structured dataRandom ForestSecondaryRobust, explainableLinear RegressionBaselineBenchmark comparisonLSTMAdvancedRequires 2+ years sequential data
Train/Val/Test Split: 70% / 15% / 15% — using TimeSeriesSplit (never shuffle)
Evaluation Metrics: MAE, RMSE, R² (target >0.85), MAPE (target <10%)

🎨 Design System
Typography
RoleFontWeightsDisplay / HeadlinesSyne400–800Body CopyLora (+ italic)400–600Metadata / UI / CodeJetBrains Mono300–500
All fonts loaded with font-display: swap and -webkit-font-smoothing: antialiased.
Type Scale — Perfect Fourth (×1.333)
All values fluid via clamp() from 320px → 1440px viewport:
css--fs-hero : clamp(3.5rem, 6vw + 1.5rem, 6rem)
--fs-2xl  : clamp(2.369rem, 3.5vw + 1rem, 3.157rem)
--fs-xl   : clamp(1.777rem, 2.5vw + 1rem, 2.369rem)
--fs-lg   : clamp(1.333rem, 1.5vw + 1rem, 1.777rem)
--fs-base : clamp(1rem, 0.5vw + 0.875rem, 1.125rem)
--fs-sm   : clamp(0.833rem, 0.5vw + 0.72rem, 0.875rem)
Color Tokens
Accent (Green — food/sustainability theme):
css--color-accent:       hsl(142, 65%, 40%)
--color-accent-hover: hsl(142, 70%, 32%)
--color-accent-bg:    hsl(142, 80%, 95%)
--color-accent-text:  hsl(142, 70%, 25%)
Status Colors:
cssGreen  → hsl(142, 65%, 40%)   /* Normal demand */
Amber  → hsl(38,  95%, 50%)   /* High demand   */
Red    → hsl(4,   80%, 55%)   /* Critical demand */
CSS Architecture
css@layer reset, tokens, base, layout, components, utilities, animations;

No !important anywhere
No inline styles (except JS-generated dynamic values)
CSS custom properties for all design tokens
8px baseline grid (--sp-1 through --sp-24)

Dark Mode

System: @media (prefers-color-scheme: dark)
Manual: [data-theme="dark"] on <html>
Inline <script> before </head> prevents flash of wrong theme
Persisted to localStorage as canteen-theme


📄 Page Sections
SectionIDDescriptionSticky Nav—Logo, nav links, dark mode toggleHero#dashboardHeadline, lead copy, CTAs, stats dlDemand Predictor#predictorInteractive input panel + results card7-Day Forecast#forecastFull week table with trend arrowsHow It Works—3 feature cards with hover liftAlgorithm Explainer#algorithmProse, feature importance chart, pseudocodeToken Reference#tokensWeight token table with impact pillsPrediction History#historyLive Firestore feed of last 5 predictionsFooter—3-column nav, copyright, tagline

🚀 Getting Started
Prerequisites

A modern browser (Chrome, Firefox, Safari, Edge)
A Firebase project (already configured — see Firebase Setup)
Internet connection (for Google Fonts + Firebase SDK CDN)

Run Locally
bash# Clone the repository
git clone https://github.com/YOUR_USERNAME/canteenai.git
cd canteenai

# Option 1: Open directly
open index.html

# Option 2: Serve with a local dev server (avoids CORS issues)
npx serve .
# or
python3 -m http.server 8080
# then open http://localhost:8080

Note: Opening index.html directly via file:// protocol may block Firebase requests in some browsers. Use a local server for the full experience.

Customization
To change the Firebase project, update the config object in index.html:
jsconst firebaseConfig = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT.firebaseapp.com",
  projectId:         "YOUR_PROJECT_ID",
  storageBucket:     "YOUR_PROJECT.firebasestorage.app",
  messagingSenderId: "YOUR_SENDER_ID",
  appId:             "YOUR_APP_ID",
  measurementId:     "YOUR_MEASUREMENT_ID"
};

🌍 Deployment
GitHub Pages
bash# Push to GitHub, then enable Pages:
# Settings → Pages → Source: Deploy from branch → main → / (root)
Your site will be live at: https://YOUR_USERNAME.github.io/canteenai/
Important: Rename canteen-ai.html to index.html for GitHub Pages to serve it automatically.
Netlify
bash# Drag and drop the folder on netlify.com/drop
# or connect your GitHub repo for auto-deploy on push
Vercel
bashnpm i -g vercel
vercel
# Follow the prompts — no configuration needed
Firebase Hosting
bashnpm install -g firebase-tools
firebase login
firebase init hosting
# Set public directory to: . (current folder)
# Configure as single-page app: No
firebase deploy

🗃 Firestore Data Schema
Every time "Predict Demand" is clicked, a document is written to the predictions collection:
json{
  "timestamp":     "<Firestore ServerTimestamp>",
  "day":           4,
  "dayName":       "Friday",
  "meal":          "lunch",
  "weather":       "rainy",
  "event":         "exam",
  "confidence":    95,
  "totalPortions": 847,
  "results": [
    {
      "name":     "Dal Rice",
      "portions": 312,
      "conf":     94,
      "status":   "red"
    },
    {
      "name":     "Biryani",
      "portions": 248,
      "conf":     93,
      "status":   "amber"
    },
    {
      "name":     "Sandwich",
      "portions": 119,
      "conf":     92,
      "status":   "green"
    },
    {
      "name":     "Samosa",
      "portions": 98,
      "conf":     91,
      "status":   "green"
    },
    {
      "name":     "Chai",
      "portions": 70,
      "conf":     90,
      "status":   "green"
    }
  ]
}
Firestore Collection: predictions
Query used for history: orderBy('timestamp', 'desc').limit(5)

📈 Analytics Events
Event NameTriggerParameterspage_viewOn page loadpage_title: 'CanteenAI Predictor'prediction_runOn "Predict Demand" clickmeal, weather, event, confidence
View events in: Firebase Console → Analytics → Events
Or: Google Analytics 4 → Reports → Events

♿ Performance & Accessibility
Accessibility

Semantic HTML5: <header>, <nav>, <main>, <section>, <article>, <footer>, <dl>, <figure>
aria-label on all nav and interactive elements
aria-labelledby linking sections to headings
aria-pressed on all toggle buttons
role="list" on styled lists
WCAG AA 4.5:1 minimum contrast on all text
::selection styled with accent colors
prefers-reduced-motion disables ALL animations

Performance

Zero npm dependencies — no node_modules, no build step
Single file — one HTTP request for the entire app
CDN fonts with font-display: swap
Firebase SDK via CDN — loaded from Google's global CDN
CSS @layer — predictable cascade, no specificity wars
IntersectionObserver — lazy-triggers animations only when visible
{ passive: true } on scroll listeners


🗺 Roadmap

 Firebase Authentication — user login to scope predictions per canteen manager
 Real ML Model — replace JS simulation with a Python Flask/FastAPI backend running real XGBoost
 CSV Export — download weekly forecast as spreadsheet
 Push Notifications — alert managers when critical demand predicted for next meal
 Multi-Canteen Support — Firestore sub-collections per canteen location
 Occupancy Input — add live attendance count as input factor
 Historical Chart — line chart of last 30 days actual vs predicted
 PWA — service worker + manifest for offline support and home screen install
 Admin Dashboard — Firestore-backed analytics with waste tracking over time


🤝 Contributing
Contributions are welcome! Please follow these steps:
bash# 1. Fork the repo
# 2. Create your feature branch
git checkout -b feature/your-feature-name

# 3. Make your changes to index.html

# 4. Test in multiple browsers

# 5. Commit with a descriptive message
git commit -m "feat: add occupancy input to predictor panel"

# 6. Push and open a Pull Request
git push origin feature/your-feature-name
Commit Convention
feat:     new feature
fix:      bug fix
style:    CSS/design changes
refactor: code restructuring
docs:     documentation updates
chore:    maintenance tasks

📜 License
MIT License

Copyright (c) 2025 CanteenAI

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.

🙏 Acknowledgements

Google Firebase — Firestore + Analytics backend
Google Fonts — Syne, Lora, JetBrains Mono
XGBoost — inspiration for the weighted multi-factor model architecture
Perfect Fourth Type Scale — modular typographic rhythm
