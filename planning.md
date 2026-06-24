# TakeMeter: Planning Document
**Project:** AI 201 Project 3  
**Community:** r/hiphopheads  
**Model:** Fine-tuned DistilBERT text classifier

---

## 1. Community

I chose r/hiphopheads because it is one of the most active music discussion communities on Reddit, with a high volume of text-heavy posts that vary significantly in quality and substance. The community discusses new releases, artist comparisons, industry news, leaks, and album retrospectives daily.

This community is a strong fit for a classification task because the same topic (e.g. "Is Kendrick better than Drake?") can produce posts ranging from a one-line assertion to a multi-paragraph structured argument. That variance is exactly what makes the discourse interesting to classify — the distinction between a reasoned take and a pure opinion matters to people who actually participate in the community.

The challenge is that most discourse is inherently subjective. The classification task is therefore not about whether a claim is true, but about whether the post *supports* its claim with specific references to the music.

---

## 2. Labels

### `analysis`
A post that makes a structured argument backed by specific evidence. The evidence can include quoted lyrics, references to production details, comparisons to other projects, historical context, or traceable reasoning. The key test: if you removed the opinion framing, would a real argument still be standing?

**Example 1:**  
"GKMC is Kendrick's best album because the narrative structure is airtight — every track advances the story arc from Compton to the pool party. The sequencing isn't random, it mirrors a classic three-act structure."

**Example 2:**  
"People sleep on Freddie Gibbs as a top-5 lyricist. His cadence switches on Bandana are technically impressive — he shifts flow mid-bar without losing the pocket, which almost no one does consistently."

---

### `hot_take`
A bold, confident opinion stated without supporting evidence. The claim might be true, but the post asserts rather than argues. There is no cited evidence, no structured reasoning — just the opinion delivered with confidence.

**Example 1:**  
"Drake hasn't made a good album since Take Care. Everything after that is just commercial filler."

**Example 2:**  
"Nicki Minaj is still the best female rapper alive and it's not close. Nobody touches her verse catalog."

---

### `reaction`
An immediate emotional response to a specific piece of music, release, news, or event. Little to no argument — the post is expressing how something made the person feel, not evaluating it critically. Common in new release threads, leak threads, and breaking news posts.

**Example 1:**  
"Just heard the new Tyler album, I am not okay. This man does not miss."

**Example 2:**  
"The leak dropped and I've had it on repeat for 3 hours. I can't even form words right now."

---

## 3. Hard Edge Cases

**The main boundary: `hot_take` vs `analysis`**

The hardest posts to label are ones that include one piece of evidence — a single lyric quote, one stat, one comparison — but are otherwise structured as a bold assertion.

*Example ambiguous post:*  
"Kendrick is overrated — his only truly great album is TPAB and everything else is inconsistent."

This names a specific album and makes a comparative claim, but it doesn't actually argue *why* TPAB is great or *why* the others are inconsistent. It's a supported-sounding assertion, not a real argument.

**Decision rule:** If the post provides specific, verifiable evidence that would support the claim *even if you removed the opinion framing*, label it `analysis`. If the evidence is vague, cherry-picked, or decorative — just enough to sound credible but not genuinely reasoned — label it `hot_take`. One lyric quote alone does not make a post `analysis`.

**Secondary boundary: `reaction` vs `hot_take`**

Short posts like "this album is trash" could be either. Decision rule: if the post is responding to something that just dropped or just happened (new release, breaking news, leak), default to `reaction`. If it's a standalone opinion not tied to a specific recent event, label it `hot_take`.

---

## 4. Data Collection Plan

**Source:** r/hiphopheads — post titles and body text from discussion threads, AOTY (Album of the Year) threads, and weekly discussion posts.

**Collection method:** Manual copy-paste into a CSV with columns: `text`, `label`, `notes`.

**Target distribution:**
- `analysis`: ~70 examples (~35%)
- `hot_take`: ~80 examples (~40%)
- `reaction`: ~60 examples (~30%)

`analysis` posts will require the most active hunting — they appear in album deep-dive threads, "unpopular opinion" threads, and long comment chains rather than top-level posts.

**If a label is underrepresented after 150 examples:** Search specifically for that label type. For `analysis`, search r/hiphopheads for "because," "breakdown," "production," or "lyrically." For `reaction`, check new release megathreads.

**Split:** The Colab notebook handles this automatically at 70% train / 15% validation / 15% test.

---

## 5. Evaluation Metrics

**Primary metric: per-class F1 score**

Accuracy alone is not sufficient because the dataset may have mild class imbalance. A model that predicts `hot_take` for everything could achieve decent accuracy but be useless. F1 balances precision and recall per class, which gives a clearer picture of whether the model has actually learned each distinction.

**Secondary metrics:**
- Confusion matrix — to identify which label pairs the model confuses most
- Per-class precision and recall — to understand whether errors are false positives or false negatives

**Baseline comparison:** Zero-shot Groq Llama-3.3-70b-versatile on the same test set.

---

## 6. Definition of Success

A classifier that is genuinely useful would need to correctly identify `analysis` posts with F1 ≥ 0.65, since that is the hardest and most valuable category to detect. Overall accuracy above 70% on a 3-class task would be meaningful (random baseline is 33%).

A model that scores well on `reaction` and `hot_take` but fails completely on `analysis` (F1 ≈ 0) would not meet the success bar — it would have missed the most interesting distinction.

The fine-tuned model should meaningfully outperform the zero-shot baseline, especially on `analysis`, where task-specific training should help the most.

---

## 7. AI Tool Plan

### Label stress-testing
Before annotating 200 examples, I will paste my label definitions and edge case description into Claude and ask it to generate 8–10 posts that sit on the boundary between `hot_take` and `analysis`. If I cannot cleanly classify the generated posts, my definitions need tightening.

### Annotation assistance
I may use an LLM to pre-label a batch of 50 examples by providing my label definitions and asking it to assign one label per post. I will review and correct every pre-assigned label before committing it to the dataset. Any pre-labeled examples will be disclosed in the AI usage section of the README.

### Failure analysis
After fine-tuning, I will paste my list of wrong predictions into Claude and ask it to identify any systematic patterns — e.g. "the model consistently misclassifies short posts" or "it confuses `hot_take` and `analysis` when the post includes a proper noun." I will verify any identified patterns by re-reading the examples myself before including them in the evaluation report.
