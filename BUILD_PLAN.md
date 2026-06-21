# Build Plan — Thought Galaxy
**Team:** Medha (A · pipeline), Linda (B · galaxy UI), Gargi (C · infra + agents)
**Deadline:** Sunday 11am submission, judging 1-3pm
**Current time:** ~6pm Saturday. ~15 hours left, ~11 productive.

---

## RIGHT NOW — 6pm to 7pm · Everyone: get running

Do these in parallel. Don't help each other until your own machine is working.

**Medha (A)**
```bash
cd backend
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp ../.env.example .env   # fill in DEEPGRAM_API_KEY + ANTHROPIC_API_KEY now
uvicorn app.main:app --reload
```
→ hit localhost:8000/health → must return {"ok":true} before moving on

**Linda (B)**
```bash
cd frontend
npm install
npm run dev
```
→ open localhost:5173 AND your mindmap.html side by side
→ your job all night is making the React app look + feel like your HTML

**Gargi (C)**
- Create Redis Cloud free account → get connection string → paste as REDIS_URL in .env
- Get Sentry DSN + Arize keys → paste into .env
- Run: `python -c "from dotenv import load_dotenv; load_dotenv(); from app.memory import ensure_index; ensure_index(); print('Redis OK')"`

---

## 7pm to 9pm · Build in parallel, no merging yet

### Medha (A) — Deepgram → terminal
One job: speak into mic, see classified JSON nodes printed in terminal.
- Test `deepgram_stream.py` directly first — just get words appearing
- Then add the Claude classify call — paste a transcript, get nodes back
- Do NOT touch the WebSocket yet. Terminal only.

```bash
# quick test once backend is running:
curl -X POST http://localhost:8000/classify \
  -H "Content-Type: application/json" \
  -d '{"transcript": "I have a CS midterm friday and im stressed, also need to email my professor about an extension"}'
```
→ should return a session with 2-3 nodes. If it does, you're done for this block.

### Linda (B) — Port blob bubbles into Galaxy.jsx
Reference your mindmap.html the whole time. Specifically port:
- `blobPath()` function → add to Galaxy.jsx
- `drawNode()` with the glow + sparkle dots → replace the basic D3 circles
- Your color system: the organic palette, opacity, shadow blur
- The gentle `alphaDecay` drift so it feels alive

Don't worry about cluster zoom yet. Just get blobs rendering where circles are.

### Gargi (C) — Wire Redis to actually save sessions
Right now `main.py` saves sessions in a try/except that silently fails. Make it real:
- Confirm `ensure_index()` passes
- Call `save_session()` with a fake session object and confirm the key appears in Redis Cloud dashboard
- Then confirm `get_session()` retrieves it back
- Add `OPENAI_API_KEY` to .env (needed for real embeddings in memory.py — free tier is fine)

---

## 9pm · SYNC #1 (15 minutes, everyone stops)

Show each other:
- Medha: curl /classify returns clean nodes ✓
- Linda: blobs visible on canvas in React ✓
- Gargi: Redis save/retrieve confirmed ✓

If all three work → wire Medha into Linda's canvas now.
If any are broken → all three debug together before moving on.

---

## 9pm to 11pm · Milestone 1 (wire everything together)

### Medha (A) + Linda (B) — Connect WebSocket to canvas
- Medha: confirm WebSocket in main.py sends `{type:'nodes', nodes:[...]}` on utterance end
- Linda: connect `useRecorder.js` to the WebSocket, feed nodes into Galaxy component
- Speak → words appear as partial text → pause → blobs bloom on canvas

This is your demo moment. Spend real time here. Speak messy rambles and tune the Claude prompt in `classify.py` until the nodes feel right.

### Gargi (C) — Tune Claude prompt + start Fetch.ai
- Help Medha test different rambles against the classifier — you're the prompt tuner
- Once Redis is confirmed, start `python agents/insight_agent.py`
- Copy the printed agent address into .env as INSIGHT_AGENT_ADDRESS
- Register on Agentverse (agentverse.ai) — this is what earns Fetch.ai prize

---

## 11pm · MILESTONE 1 CHECKPOINT 🚨

**STOP BUILDING. Do these three things first:**

1. Demo it to each other — speak 60 seconds, blobs appear, connections draw
2. **Create Devpost draft RIGHT NOW** — add all three teammates, project name, one line description. Even blank. This guarantees judging.
3. Commit and push everything to GitHub

You are now demoable. You can win from here. Don't forget this.

---

## 11pm to 1am · Milestone 2

### Medha (A) — /suggest endpoint
The endpoint already exists in main.py. Make it actually work:
- `suggest.py` calls `get_session()` from Redis → pulls the tapped node → calls Claude → returns one suggestion
- Test with curl:
```bash
curl -X POST http://localhost:8000/suggest \
  -H "Content-Type: application/json" \
  -d '{"node_id": "YOUR_NODE_ID", "session_id": "YOUR_SESSION_ID"}'
```
→ should return a warm, grounded suggestion sentence

### Linda (B) — Guidance card UI
- Tap a bubble → card slides in from right
- Shows node type chip + text + suggestion from /suggest
- "drew on past sessions" line at bottom if Redis returned past context
- Close button
- Reference the card styles already in styles.css

### Gargi (C) — Route /suggest through Fetch.ai Insight Agent
Instead of suggest.py calling Claude directly, route through the uAgent:
- Insight Agent is already written in `agents/insight_agent.py`
- Update `suggest.py` to send a message to the agent address and await reply
- This is the difference between "we used Claude" and "we have a multi-agent system" — judges notice

---

## 1am · MILESTONE 2 CHECKPOINT

Tap a bubble → guidance card appears → suggestion references something real.
Commit. Push. You're done building.

---

## 1am to 2am · Wrap up

- **Everyone:** record a 2-minute backup demo video on someone's phone. Non-negotiable. Wifi WILL be sketchy during judging.
- Pre-load 2-3 past sessions with good data so M2 suggestions feel meaningful during the demo
- Fix any obvious visual bugs
- Sleep

---

## 8am to 10:30am · Submission push

| Task | Who |
|---|---|
| Devpost writeup — problem, solution, tech stack, what you learned | Gargi |
| Screenshots of the galaxy looking beautiful | Linda |
| Upload backup demo video | Medha |
| Rehearse 4-minute pitch out loud together, twice | Everyone |
| Make sure all three are at your table by 1pm for judging | Everyone |

**Devpost submission deadline: 11am. Do NOT miss this.**

---

## The 4-minute pitch

**(0:00 - 0:30) The problem — don't show screen yet**
"You come home overwhelmed. Your brain is full — midterm Friday, behind on homework, a friend situation you haven't dealt with. No app helps with that. Notion wants you to organize. Todoist wants a list. But your brain doesn't think in lists."

**(0:30 - 2:00) Live demo M1 — say nothing, let it breathe**
Talk through a real messy day. Let the bubbles bloom live. Let the connections draw. The silence while the galaxy builds itself IS the pitch.

**(2:00 - 2:45) M2 — the insight moment**
Tap a stress bubble. "What should I do?" Show the suggestion card. "It's pulling from what I told it last week. It knows this isn't the first time."

**(2:45 - 3:20) Tech stack — fast**
"Deepgram handles the voice. Claude classifies every thought in real time. Fetch.ai agents handle the reasoning and execution. Redis remembers everything across sessions."

**(3:20 - 4:00) Vision**
"This is a map of your inner life. And unlike every other productivity app, it doesn't ask you to change how you think. You just talk."

**If anything breaks live:** `/classify` REST endpoint is your fallback — type instead of speak. Backup video is your floor.

---

## Sponsor prize checklist

| Sponsor | What you did | How to claim |
|---|---|---|
| **Deepgram** | Voice is the entire input layer | Works automatically — mention in writeup |
| **Anthropic** | Claude classifies every thought + powers suggestions | Mention Claude Code in writeup |
| **Fetch.ai** | Insight Agent routes suggestions via Agentverse | Register agent, show it's discoverable |
| **Redis** | Saves sessions + semantic search across past thoughts | Show Redis dashboard to judges |
| **Arize** | Logs every classification decision | Screenshot dashboard for judges |
| **Sentry** | Error monitoring across stack | Just keep DSN set, mention in writeup |

Target: **6 sponsor prizes** from one coherent app.

---

## Track

**Ddoski's World** — frame it as mental clarity and emotional organization access.
"Every productivity app assumes you already know what you need to do. This one helps you figure it out."

Also gunning for: **Best UI/UX** (the galaxy), **Golden Bear** (frame around Berkeley student life), **Hacker's Choice** (the live demo wins hearts).
