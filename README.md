# Fluent IRL — MVP

AI-powered English fluency practice. Roleplay real-life social scenes (party, coffee shop, gym) with a human-like AI partner.

> Built for social fluency, not grammar drills.

---

## File structure

```
fluent-irl/
├── frontend/                       # React Native (Expo) app
│   ├── App.js
│   ├── app.json
│   ├── babel.config.js
│   ├── package.json
│   ├── .env.example
│   └── src/
│       ├── navigation/
│       │   └── AppNavigator.js
│       ├── screens/
│       │   ├── OnboardingScreen.js
│       │   ├── HomeScreen.js
│       │   ├── ChatScreen.js
│       │   └── FeedbackScreen.js
│       ├── components/
│       │   ├── ScenarioCard.js
│       │   ├── ChatBubble.js
│       │   ├── TypingIndicator.js
│       │   └── HelpSheet.js
│       ├── services/
│       │   ├── api.js              # backend client (mock fallback)
│       │   ├── supabase.js
│       │   └── voice.js            # mic recording hook
│       ├── data/
│       │   └── mockData.js
│       └── theme/
│           └── theme.js            # dark-mode tokens
│
├── backend/                        # Node.js + Express
│   ├── package.json
│   ├── .env.example
│   └── src/
│       ├── server.js
│       ├── routes/
│       │   ├── chat.js             # POST /api/chat
│       │   ├── help.js             # POST /api/help
│       │   └── feedback.js         # POST /api/feedback
│       ├── services/
│       │   ├── llm.js              # Anthropic + OpenAI abstraction
│       │   └── supabase.js
│       └── prompts/
│           └── prompts.js          # AI personalities, scenarios, levels
│
└── database/
    └── schema.sql                  # Supabase tables + RLS
```

---

## Setup

### 1. Database

In Supabase SQL editor, run `database/schema.sql`.

Enable **Anonymous sign-ins** in Auth → Settings (the onboarding flow uses them so users can try the app before creating an account).

### 2. Backend

```bash
cd backend
cp .env.example .env
# fill in either ANTHROPIC_API_KEY or OPENAI_API_KEY
# fill in SUPABASE_URL + SUPABASE_SERVICE_ROLE_KEY
npm install
npm run dev
```

Backend runs on `http://localhost:3000`.

### 3. Frontend

```bash
cd frontend
cp .env.example .env
# fill in EXPO_PUBLIC_SUPABASE_URL, EXPO_PUBLIC_SUPABASE_ANON_KEY
# set EXPO_PUBLIC_API_URL=http://YOUR-LAN-IP:3000   (use your LAN IP, not localhost, when testing on a real device)
npm install
npm start
```

> If you don't set `EXPO_PUBLIC_API_URL`, the app runs in **mock mode** so you can preview the UI without the backend.

---

## How the AI stays human

The whole product hinges on the AI not sounding like a chatbot. The prompt system in `backend/src/prompts/prompts.js` does three things:

1. **Scenario context** — sets the scene, persona, and tone (party = casual/slangy, coffee = quick/efficient, gym = short/low-energy).
2. **Random personality per conversation** — friendly, busy, cold, interested, distracted. Forces the user to handle realistic variation.
3. **Level adaptation** — the AI matches the user's vocabulary level so they can keep up, but never corrects grammar mid-conversation.

Hard rules baked into the system prompt:
- Never break character.
- Never explain English.
- Never say "Certainly!" or "I'd be happy to..."
- Keep replies 1–2 sentences.
- React emotionally (laugh, get bored, push back).

Feedback only happens **after** the conversation ends, on the dedicated Feedback screen — so the practice itself stays immersive.

---

## API endpoints

| Method | Path           | Purpose |
|--------|----------------|---------|
| POST   | /api/chat      | Send the conversation history, receive the AI's next reply in character. |
| POST   | /api/help      | Get 3 natural reply suggestions for the user's next turn. |
| POST   | /api/feedback  | End-of-conversation: returns naturalness score (0–100), highlights, and suggested rephrasings. |
| GET    | /health        | Health check. |

All three accept JSON. Schemas in the route files.

---

## What's intentionally NOT in the MVP

- Speech-to-text (the mic button records audio; wire it to Whisper/Deepgram later — `voice.js` returns the URI).
- Streaming AI responses (use SSE in v2 to make replies feel even faster).
- Per-user progress dashboard (the `progress` table is being populated — build the screen next).
- Auth UI (anonymous sign-in is good enough to validate the loop).

---

## Suggested build order

1. Run frontend in mock mode → walk through onboarding → home → chat → feedback. Confirm the UX feels right.
2. Stand up the backend with one provider key. Switch `EXPO_PUBLIC_API_URL` on. Test the chat loop.
3. Apply Supabase schema. Test that conversations + messages persist.
4. Add Whisper for voice input.
5. Add a progress screen reading from the `progress` table.
