# PASS/SCAN — Password Strength Analyzer

A single-page, client-side password strength analyzer built with vanilla HTML, CSS, and JavaScript. Type a password and get a live strength score, entropy-based stats, estimated crack time, disguised-dictionary-word detection, and a real breach check against the Have I Been Pwned database.

[Live Demo] ← https://arsheri07.github.io/password-strength-analyzer/

## Why this project

Password strength meters are one of the most common security UX patterns on the web, and building one from scratch is a practical way to demonstrate:

- Understanding of basic password security principles (entropy, character diversity, common-password/breach exposure)
- Client-side validation logic and regex
- Thoughtful UX for security tooling (immediate feedback, no dark patterns, no data leaving the browser)

## Features

- **Live scoring (0–100)** as the user types, no submit button required
- **Entropy-based strength math** — score is driven primarily by Shannon entropy (bits), not just a checklist of character types
- **Estimated crack time**, shown for both an offline attack (fast GPU cracking a leaked hash) and an online attack (a rate-limited login form) — makes the abstract score concrete
- **Real breach detection** via the [Have I Been Pwned Pwned Passwords API](https://haveibeenpwned.com/API/v3#PwnedPasswords), using its k-anonymity model: only the first 5 characters of a SHA-1 hash are sent, so the real password never touches the network
- **Dictionary + l33t-speak detection** — normalizes common substitutions (`@`→a, `0`→o, `3`→e, `$`→s, etc.) and checks the result against a list of common words, names, and passwords, so `P@ssw0rd123` is flagged even though its raw entropy score looks decent on paper
- **Pattern checks**: repeated characters (`aaaa`) and obvious sequences (`1234`, `abcd`), on top of the standard length/case/number/symbol rules
- **Weak-password alert** — a native browser alert fires once per weak/compromised entry, including the estimated crack time, without being spammy
- **Debounced network calls** — the breach check waits for a pause in typing rather than firing on every keystroke
- **Show/hide toggle** so users can verify what they typed
- **Privacy-conscious by design** — no analytics, no storage, no logging; the only outbound request is the 5-character hash prefix sent to HIBP

## How the scoring works

1. **Entropy** is calculated as `length × log2(character pool size)`, where the pool size depends on which character classes are present (lowercase, uppercase, digits, symbols). This is the same core idea real strength meters use — more possible characters and more length both increase the search space an attacker has to brute-force.
2. **Crack time** is estimated from that search space divided by an assumed guesses/second rate — `10 billion/sec` for an offline attack against a leaked hash, and `~100/hour` for a rate-limited login form. Both assumptions are stated directly in the code so the numbers are explainable, not mysterious.
3. **Rule checks** (length, character classes, no repeats/sequences, no disguised dictionary words) contribute a smaller 30% weight on top of the entropy score, mostly to catch patterns that raw entropy math under-penalizes.
4. **A confirmed breach match caps the score at 10**, and a **disguised dictionary word caps the score at 35**, regardless of entropy — a password everyone already knows (or could easily guess with a wordlist) is unsafe no matter how "complex" the math says it is.

This is intentionally still simpler than a production library like [zxcvbn](https://github.com/dropbox/zxcvbn), which ships tens of thousands of dictionary entries and models keyboard-walk patterns far more exhaustively — see Limitations below.

## Tech stack

- HTML5
- CSS3 (custom properties, no frameworks)
- Vanilla JavaScript (no dependencies)

Zero build step — open `index.html` in any browser and it works.

## Running locally

```bash
git clone https://github.com/<your-username>/password-strength-analyzer.git
cd password-strength-analyzer
open index.html   # or just double-click the file
```

No installs, no server required.

## Limitations & honest tradeoffs

Being upfront about these in the README is itself a signal of engineering maturity to anyone reviewing your repo:

- The dictionary word list is a small hardcoded sample (~70 entries) for demonstration. Production libraries like [zxcvbn](https://github.com/dropbox/zxcvbn) use much larger corpora and more sophisticated pattern matching (keyboard walks, date patterns, name+number combos).
- The crack-time numbers depend entirely on the stated guesses/second assumptions, which are order-of-magnitude estimates, not measured benchmarks — real attacker hardware varies widely.
- `crypto.subtle` (used for the SHA-1 hashing in the breach check) requires a secure context — it works over HTTPS (like GitHub Pages) but may not work if you open the file directly from disk in some browsers.
- The HIBP breach check requires an internet connection; if it fails, the tool falls back to entropy/rule-based scoring only and says so explicitly rather than silently guessing.
- This is a teaching/portfolio tool, not a vetted security product — it shouldn't be the only gate in front of a real signup form.

## Ideas for extending this project

- Expand the dictionary word list significantly, or swap the whole detection engine for Dropbox's `zxcvbn` library and compare its output against this one
- Add a password generator that suggests a strong alternative when the current one scores poorly
- Add unit tests (Jest) for the entropy math, sequence detection, and dictionary-matching functions
- Show a running history of scores as the user edits, as a mini visualization
- Let the guesses/second assumptions be adjustable via a dropdown ("assume a nation-state attacker" vs "assume a rate-limited form") to make the tradeoff more interactive

## License

MIT — feel free to fork and adapt for your own portfolio.

