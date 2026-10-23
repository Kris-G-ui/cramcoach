# CramCoach 📚⚡

**An AI study buddy that turns your notes into a study pack and an adaptive quiz in seconds.**

CS4680 Prompt Engineering · AI Assistant Project

🔗 **Live demo:** https://Kris-G-ui.github.io/cramcoach/
🎬 **Demo video:** YOUR-YOUTUBE-LINK

---

## The problem

Students often study the night before an exam with a pile of disorganized lecture notes. Re-reading notes is one of the least effective ways to study; **active recall** (testing yourself) and **targeted review of mistakes** work far better. But writing good practice questions takes hours, and generic AI chatbots give inconsistent, unstructured answers that are hard to study from.

## The solution

CramCoach takes whatever you have (pasted notes, a textbook section, or just a topic name) and in a few seconds produces:

- **Summary** – a plain-language overview pitched at your level
- **Key concept flashcards** – tap to flip
- **Memory trick** – a mnemonic or analogy for the hardest idea
- **Interactive quiz** – multiple choice with instant feedback and an explanation for every answer
- **Adaptive follow-up** – one click to "Drill what I missed," "Make it harder," "Explain simpler," or any custom request; CramCoach rewrites the pack around your actual mistakes

A built-in **Prompt Inspector** shows the exact prompts sent to the model, so you can see the prompt engineering working.

## How it works

```
 Student input ──► Prompt builder ──► Gemini API ──► JSON ──► Study pack UI
 (notes, level,     (persona +          (JSON mode)              │
  quiz length,       few-shot +                                  ▼
  goal)              template)                         Student takes quiz
                         ▲                                       │
                         └──── refinement prompt + wrong answers ◄┘
```

1. The student pastes material and picks a level, quiz length and optional goal.
2. The app builds a conversation: **system persona** → **few-shot example pair** → the student's request wrapped in a **structured template** with an exact JSON schema.
3. Gemini returns JSON (enforced with `responseMimeType: "application/json"`), which the app validates and renders as interactive UI.
4. When the student asks for a change, the app appends a **refinement prompt** containing their feedback *and the questions they got wrong* to the same conversation, so the model revises its own work instead of starting from scratch.

## Prompt engineering techniques

CramCoach combines four patterns. All prompts live at the top of the `<script>` in `index.html`.

### 1. Persona pattern
The system instruction makes the model **"CramCoach, an upbeat, patient tutor who has helped thousands of students pass exams the night before."** Beyond tone, the persona carries teaching rules: correct mistakes in the student's notes, match depth to their level, test understanding instead of trivia, and write wrong answers that reflect real misconceptions.

**Why it helps:** Because it's sent as a system instruction, the persona applies to every turn, including refinements, so tone and quality stay consistent. Without it, responses varied between overly casual and textbook-dry, and quiz distractors were often obviously wrong.

### 2. Few-shot pattern
Before the student's request, the app sends one complete worked example: a "Photosynthesis" request and an ideal response. The example demonstrates distractors based on real misconceptions (e.g. "carbon dioxide" as the source of released oxygen), correct answers in varied positions, and explanations that cover why the right answer is right *and* why the tempting wrong one is wrong.

**Why it helps:** Rules describe what we want; an example *shows* it. The example sets the quality bar far more reliably than adding more instructions. It is placed as a real user/model turn pair so the model treats it as its own past behavior to imitate.

### 3. Template pattern (structured output)
Every request uses the same template: labeled fields (`STUDENT LEVEL`, `QUIZ LENGTH`, `MATERIAL TYPE`), the material in triple-quote delimiters, and an exact JSON schema for the output. JSON mode is also enabled in the API call.

**Why it helps:** The UI needs predictable data, not free text. The template turns the model's reply into flashcards, quiz buttons and scores. The delimiters keep pasted notes separate from instructions, and the app still validates every field, skipping malformed questions instead of crashing.

### 4. Iterative refinement pattern
After the quiz, the refine buttons send a follow-up in the **same conversation** containing the student's request, their score, and each question they missed with the answer they chose and the correct answer. The model is told to keep the format, write all-new questions, and reteach the specific misunderstandings.

**Why it helps:** This makes the app adaptive. "Drill what I missed" produces a new pack focused on exactly the ideas the student got wrong, which a one-shot prompt can't do. The version badge (Version 1 → Version 2 · refined) makes the loop visible.

### Bonus guardrails
- An **error contract**: if the input is empty or not studyable, the model returns `{"error": "..."}`, which the app shows as a friendly message.
- **Self-correction**: the persona tells the model to fix mistakes in the student's notes rather than repeat them.

## Tech stack

- **Frontend:** a single `index.html` with vanilla HTML, CSS and JavaScript (no build step)
- **AI:** Google Gemini API (`generateContent`, JSON mode)
- **Hosting:** GitHub Pages

## Run it yourself

1. Get a free Gemini API key at [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
2. Open the live demo (or open `index.html` in any browser).
3. Click **Add API key** in the top-right, paste your key and save.
4. Paste notes or click an example topic, then hit **Build my study pack**.

Your key is stored only in your browser's local storage and sent only to Google's API. It is never in this repository.

> If you get a "model not available" message, open settings and choose a different model.

## Impact and future work

**Impact:** CramCoach turns passive re-reading into active recall in under a minute, and the refinement loop targets each student's personal weak spots, the way a good tutor would.

**Future ideas:**
- Upload PDFs or lecture slides directly
- Spaced repetition: save missed questions and resurface them over days
- Export flashcards to Anki or Quizlet
- Track progress across topics over a semester
- Voice mode for studying on the go

## Author

YOUR NAME · CS4680 Prompt Engineering
