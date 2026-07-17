# 🧥 Customer Feedback Triage & Automated Replies

> A clothing shop gets thousands of reviews a week, and someone reads *every single one* by hand just to catch the unhappy customers and write back. This project does both of those jobs automatically: it finds the reviews that need a reply, and it drafts the reply.

**Hiral Sarkar** · Imarticus Data Science Internship *(Python Foundations & Gen AI)*
Everything lives in one notebook → [`customer_feedback_analysis.ipynb`](customer_feedback_analysis.ipynb)

---

## 🎯 The 30-second version

- **23,486** reviews go in. I clean them down to the ones that actually have text.
- **~10%** turn out to be *critical* (1 or 2 stars) — the pile that needs a human.
- Those unhappy reviews complain about the same few things, led hard by **fit** and **quality**.
- The three most urgent get a **warm, specific apology email**, drafted by a free AI model and saved as ready-to-send Word documents.

There's **no machine learning** in the filtering, because the brief asked for plain rules. That's a feature, not a limitation: every decision here is one you can read, follow, and argue with.

---

## 📊 What the data actually said

Before automating anything, I wanted a feel for the reviews. First question: how happy are people, really?

<p align="center"><img src="images/ratings.png" width="620" alt="Rating distribution"></p>

Most customers are delighted (4 and 5 stars run away with it). The two red bars are the 1 and 2 star reviews, and they're a clear minority. That's the whole reason automating *just this slice* makes sense: it's small enough to handle with care instead of drowning in.

So what are those unhappy customers actually upset about? I grouped the common complaint words into themes by hand:

<p align="center"><img src="images/themes.png" width="620" alt="Complaint themes"></p>

> 💡 **The headline:** complaints cluster hard around **fit/sizing** and **quality** — the exact two things you *cannot* judge from a product photo. That's not a vague "customers are unhappy", it points the buying team at a real, fixable problem.

---

## 🤖 The payoff: a reply the model actually wrote

Here is one of the three drafts, generated for a real 1-star review about a kimono whose fabric was ripping at the neckline. Notice it names the *specific* problem and the price complaint, instead of a copy-paste "sorry for your experience":

> **Subject: We're so sorry about your kimonos**
>
> Hi there,
>
> I was truly sorry to read about the issues you're experiencing with our kimonos. It sounds incredibly frustrating to love the design and colors, only to find the fabric ripping at the neckline and developing holes. Please accept my sincerest apologies; this is not the level of quality we strive for, and it's clear we missed the mark here.
>
> You deserve garments that last, especially at that price point, and I take full responsibility for this disappointment. I want to make this right for you immediately. Would you prefer a full refund for the damaged items, or would you like us to send out high-quality replacements?
>
> Just let me know which option works best for you, and I will handle the rest.
>
> Warmly,
> Aria, Customer Care, StyleNest

Each of the three replies is also saved as its own **Word document** in [`emails/`](emails), so an agent can open it, tweak a line, and hit send.

*(The data has no real brand name — it was anonymised — so the emails needed a shop to come from. I invented one and called it **StyleNest**.)*

---

## 🧭 How it works, step by step

The notebook reads top-to-bottom like a story, each step feeding the next:

| Step | What happens | How |
|------|--------------|-----|
| **1. Clean** | Drop reviews with no text, tidy the wording | pandas + basic string work |
| **2. Explore** | See how ratings spread and where the bad ones cluster | charts |
| **3. Filter** | Pull out the critical (1–2 star) reviews | one boolean rule, no ML |
| **4. Understand** | Find the words, phrases and themes people complain about | `collections.Counter` |
| **5. Prioritise** | Pick the 3 most urgent reviews to answer first | a plain ranking rule |
| **6. Reply** | Draft a personal apology for each and save it | a free LLM via OpenRouter |

### 🔍 A peek at the complaint-mining

The single most common words tell you what's wrong at a glance:

<p align="center"><img src="images/keywords.png" width="600" alt="Top complaint words"></p>

Getting here took a little iteration. My first word count was mostly filler ("like", "back", "one"), so I kept growing a small, hand-written stopword list until the words left standing actually meant something. I also count **two-word phrases** (keeping words like "too" and "not"), which recover the context single words lose — things like *"not flattering"*, *"too small"*, *"see through"* and *"very thin"*.

---

## 🚀 Run it yourself

You need **Python 3.9+**, **Jupyter**, and a few common libraries:

```bash
pip install pandas matplotlib requests python-docx
```

Then open the notebook and **Run All Cells**. It finishes in a couple of minutes, and the dataset is already in the folder, so there's nothing to download.

### 🔑 The API key

The reply-drafting uses **[OpenRouter](https://openrouter.ai/keys)** with a **free** model, so it costs nothing. The key is a plain variable near the top of Section 7:

```python
API_KEY = "PASTE-YOUR-OPENROUTER-KEY-HERE"
```

Grab a free key from [openrouter.ai/keys](https://openrouter.ai/keys) and paste it over the placeholder to run the AI part yourself. **You don't need one just to see the results** — all three replies are already saved in the notebook's output and in the [`emails/`](emails) folder.

> ⚙️ Free models get briefly busy and return "rate limited" errors, so the code keeps a short list of models and quietly tries the next one (waiting a moment between attempts) instead of giving up on the first busy response.

---

## 🧹 A note on the cleaning

The only column I truly need is **Review Text**. If it's empty, the review is useless to me, so I drop those rows (about 845, roughly 3.6% of the data) and move on. The other small gaps, like a missing title, are harmless and I leave them.

Then I make a cleaned *copy* of the text — lowercased, no punctuation or numbers — purely for counting words, so "Great!" and "great" are treated as the same. The **original wording is kept untouched**, because I need the real thing when I write the emails.

---

## 💡 If I had more time

- Break the complaint themes down **per department**, so the findings feed the buying team, not just support.
- Add a quick automated quality check on each drafted email before a human sees it.
- Try two or three models on the same review and compare how the tone reads.

---

<p align="center"><i>Built by <b>Hiral Sarkar</b> for the Imarticus Data Science Internship assessment.</i></p>
