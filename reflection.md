

## 1. What was broken when you started?

The first time I ran the game, it looked like a working number guessing app — it loaded fine, accepted input, and showed messages — but something felt off almost immediately when I started playing. The hints were unreliable: sometimes guessing too low would show a message pointing me in the wrong direction, and the score would occasionally go *up* after a wrong guess, which made no sense. Two concrete bugs I noticed right away were that the hint messages were backwards (both "Too High" and "Too Low" branches showed the same direction text), and that starting a new game didn't actually reset everything — the old "Game over" status would persist and block me from playing again. After looking at the code more carefully, I also found that Hard difficulty had a *smaller* range than Normal (1–50 vs 1–100), meaning Hard was accidentally the easiest mode.

---

## 2. How did you use AI as a teammate?

I used Claude (by Anthropic) as my AI assistant throughout this project, sharing the full `app.py` source code and describing the strange behavior I was seeing in the game. One example of a correct AI suggestion was when I described the hints feeling random — Claude pointed directly to the `check_guess` function and explained that on every even-numbered attempt, the secret was being cast to a string, causing Python to do alphabetical comparison instead of numerical (so `"9" > "42"` evaluates to `True` because `"9"` sorts after `"4"` alphabetically). I verified this by reading the function myself, tracing through what would happen on attempt 2 vs attempt 3, and then testing in the live game using the debug panel to confirm hints matched the secret correctly after the fix. One example of a misleading suggestion was when I asked about the scoring system — Claude initially suggested the `+5` points awarded for wrong "Too High" guesses on even attempts might be intentional "partial credit" and I should leave it in. I pushed back, mapped out all the scoring branches on paper, confirmed there was no consistent design logic supporting it, and removed the bonus points.

---

## 3. Debugging and testing your fixes

I decided a bug was really fixed when two things were true: the code change made logical sense to me when I read it, and the game behaved correctly when I played through the exact scenario that had triggered the bug before. For the hint direction bug, I ran a manual test where I opened the Developer Debug Info expander (which shows the secret number) and made deliberate wrong guesses — one I knew was too low and one I knew was too high — then confirmed the hint matched the correct direction every time, including on even-numbered attempts where the bug had previously struck. For the New Game reset bug, I tested by playing until I lost, clicking New Game, and immediately checking the debug panel to confirm that `status`, `history`, `score`, and `secret` had all been properly cleared. Claude helped me think through the scoring test by suggesting I map out every possible combination of `outcome` and `attempt_number` before touching the code, which made the broken branch immediately visible without needing to run the app at all.

---

## 4. What did you learn about Streamlit and state?

In the original app, the secret number kept changing because Streamlit reruns the entire Python script from top to bottom every time the user interacts with anything — clicking a button, typing in a box, or changing a dropdown all trigger a full rerun, and without session state, `random.randint()` would generate a brand new secret on every single rerun. To explain Streamlit reruns to a friend: imagine every button click causes the whole script to restart from line 1, as if you just refreshed the page — `st.session_state` is like a small notebook that survives those restarts so your data doesn't get wiped. The fix that gave the game a stable secret was wrapping the secret generation in a check — `if "secret" not in st.session_state` — so the random number is only generated once on the very first run, and every rerun after that just reads the already-saved value instead of creating a new one. This also taught me that *every* piece of game state (attempts, score, status, history) needs the same treatment, which is why the New Game button had to explicitly reset all of them rather than just the secret.

---

## 5. Looking ahead: your developer habits

One habit I want to reuse in future projects is reading the full function before accepting any AI suggestion — in this project, mapping out the `update_score` branches on paper before touching the code saved me from incorrectly accepting the AI's "partial credit" explanation for what was actually a bug. One thing I would do differently next time is ask the AI to explain *why* a piece of code was written a certain way before asking it to fix anything, because Claude initially defended a bug as intentional design, which could have been avoided if I had first asked "what is this supposed to do?" rather than jumping straight to "is this a bug?" This project changed the way I think about AI-generated code because I came in expecting bugs to be obvious mistakes, but several of them looked intentional and even defensible at first glance — it taught me that AI-generated code needs the same skeptical review as any other code, and that the AI helping you debug it can accidentally cover up problems if you accept its first answer without pushing back.