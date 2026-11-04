# 🎵 Music Recommender Simulation

## Project Summary

In this project you will build and explain a small music recommender system.

Your goal is to:

- Represent songs and a user "taste profile" as data
- Design a scoring rule that turns that data into recommendations
- Evaluate what your system gets right and wrong
- Reflect on how this mirrors real world AI recommenders

My version, **TuneJudge 1.0**, is a content-based recommender that runs in the terminal. It loads 19 songs from `data/songs.csv`, scores each one against a user's favorite genre, favorite mood, target energy, and acoustic preference, and prints the top 5 with the reasons behind each score. I tested it with three normal profiles and four adversarial ones to find where the scoring breaks.

---

## How The System Works

Explain your design in plain language.

Real platforms like Spotify mix collaborative filtering (what similar users liked) with
content-based filtering (songs with similar attributes). Mine is content-based only, comparing
each song's attributes to one user's taste profile, prioritizing genre first and then energy and mood.

Some prompts to answer:

- What features does each `Song` use in your system
  - For example: genre, mood, energy, tempo
  
  genre, mood, energy, acousticness
- What information does your `UserProfile` store

  favorite_genre, favorite_mood, target_energy, likes_acoustic
- How does your `Recommender` compute a score for each song

  +2 for a genre match, +1 for a mood match, up to +1.5 for energy closeness (`1 - |song.energy - target_energy|`), up to +0.5 for acoustic fit
- How do you choose which songs to recommend

  score every song, sort from highest to lowest, return the top k (default 5)

**Algorithm Recipe**

```
score = 2.0 * genre_match
      + 1.0 * mood_match
      + 1.5 * (1 - |song.energy - target_energy|)
      + 0.5 * acoustic_fit   # acousticness if likes_acoustic, else 1 - acousticness
```

Max score is 5.0. Ties go to the song with closer energy.

**Expected biases**

- Over-prioritizes genre, so a great mood and energy match from another genre can lose to a weak same-genre song.
- Exact matching gives no credit for similar moods (chill vs relaxed) or genres (pop vs indie pop).
- No valence, so it can't tell sad songs from calm ones with similar energy.
- Genres with only one song in the catalog give those users few strong matches.

You can include a simple diagram or bullet list if helpful.

---

## Getting Started

### Setup

1. Create a virtual environment (optional but recommended):

   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Mac or Linux
   .venv\Scripts\activate         # Windows

2. Install dependencies

```bash
pip install -r requirements.txt
```

3. Run the app:

```bash
python -m src.main
```

### Running Tests

Run the starter tests with:

```bash
pytest
```

You can add more tests in `tests/test_recommender.py`.

---

## Sample Recommendation Output

Output of `python -m src.main` for the default pop/happy profile:

```
Loading songs from data/songs.csv...
Loaded songs: 19

============================================================
Profile: Happy pop fan
  genre=pop, mood=happy, energy=0.80, likes_acoustic=False
============================================================
1. Sunrise City by Neon Echo  [score 4.88 / 5.00]
     - genre match: pop (+2.0)
     - mood match: happy (+1.0)
     - energy 0.82 vs target 0.80 (+1.47)
     - acoustic fit (+0.41)
2. Gym Hero by Max Pulse  [score 3.78 / 5.00]
     - genre match: pop (+2.0)
     - energy 0.93 vs target 0.80 (+1.30)
     - acoustic fit (+0.47)
3. Rooftop Lights by Indigo Parade  [score 2.77 / 5.00]
     - mood match: happy (+1.0)
     - energy 0.76 vs target 0.80 (+1.44)
     - acoustic fit (+0.33)
4. Concrete Verses by Block Theory  [score 1.91 / 5.00]
     - energy 0.78 vs target 0.80 (+1.47)
     - acoustic fit (+0.44)
5. Fuego Lento by Las Brisas  [score 1.84 / 5.00]
     - energy 0.84 vs target 0.80 (+1.44)
     - acoustic fit (+0.40)
```

**Screenshot or video** *(optional)*: <!-- Insert a screenshot or demo video link here -->

---

## Experiments You Tried

Use this section to document the experiments you ran. For example:

- What happened when you changed the weight on genre from 2.0 to 0.5
- What happened when you added tempo or valence to the score
- How did your system behave for different types of users

**Weight shift: genre 2.0 → 1.0, energy 1.5 → 3.0**
- Happy pop fan: Rooftop Lights (happy indie pop) moved ahead of Gym Hero (intense pop). This felt more correct.
- High-energy but sad: the only sad song dropped out of the top 5. This felt worse.
- Rock, sad, and metal profiles ended up with the same five loud songs.
- Overall the results were different, not better. Energy started drowning out genre and mood.

**Different types of users**
- Normal profiles (pop, lofi, rock) got the right #1 song every time.
- Conflicting preferences (high energy + sad, metal + acoustic): one preference was quietly ignored.
- A genre not in the catalog (k-pop): the system fell back to mood and energy, which worked reasonably.
- Capitalized input ("Pop"): every genre and mood match was lost, and the top song dropped from 4.88 to 1.88.

Full outputs and profile comparisons are in the [model card](model_card.md).

---

## Limitations and Risks

Summarize some limitations of your recommender.

Examples:

- It only works on a tiny catalog
- It does not understand lyrics or language
- It might over favor one genre or mood

You will go deeper on this in your model card.

- **Tiny, made-up catalog:** 19 songs, and most genres have only one song.
- **Exact matching:** "indie pop" doesn't count as "pop," and "Pop" with a capital P matches nothing.
- **Genre can outweigh everything:** a song with the right genre and mood can rank #1 even when its energy is far off.
- **Filter bubble:** loud songs are all electronic and quiet songs are all acoustic, so users who want loud acoustic or quiet electronic music get pushed toward the common type.
- **No lyrics, language, or listening history,** so it can't learn from what users actually play or skip.

---

## Reflection

Read and complete `model_card.md`:

[**Model Card**](model_card.md)

Write 1 to 2 paragraphs here about what you learned:

- about how recommenders turn data into predictions
- about where bias or unfairness could show up in systems like this

A recommender doesn't "understand" music. It turns each song into a few numbers and labels, compares them to what the user said they like, and adds up points. Even so, four simple rules produced lists that felt like real recommendations: the study profile got soft lofi, the rock profile got loud rock, and each pick came with a reason. The weights are where the real decisions happen. Changing just two of them reshuffled every list, and the results were different rather than better.

Bias showed up in places I didn't expect. It came less from the scoring formula than from the data and how input is matched. Because loud songs in my catalog are all electronic, users with less common tastes were quietly ignored. Because matching is exact, typing "Pop" instead of "pop" wiped out the top recommendation. In a real app with millions of users, the same kinds of gaps could mean whole groups of listeners keep getting pushed toward whatever the data has the most of.



