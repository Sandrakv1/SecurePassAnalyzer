# SecurePass Analyzer

A privacy-first password strength analyzer and generator built with Django. All analysis runs in the browser.

## Abstract
SecurePass Analyzer is a web-based cybersecurity tool that estimates password strength without transmitting the password. A client-side rule engine (optionally combined with the zxcvbn estimator) scores passwords from 0 to 100, flags common passwords, sequences, repeats and personal details, and gives recommendations. A generator uses the browser's cryptographically secure random source. The Django back end serves pages only and exposes no endpoint that accepts passwords; a strict Content Security Policy blocks network calls from page scripts.

## Problem statement
Composition rules ("one capital, one digit") are poor predictors of strength, and many online checkers receive the passwords users type. This project evaluates predictability and length locally.

## Objectives
Accurate and explainable scoring; zero password transmission or storage; secure generation; a clear, accessible interface.

## Features
Real-time analysis, show/hide, clear, 9-item pass/fail checklist, weaknesses and recommendations, entropy and offline guessing-time estimate with stated assumptions, optional personal-details check, generator (8–64 characters, 4 character sets, copy button, strength rating), Security Tips and About pages.

## Stack
Python, Django, HTML/CSS/JavaScript, Bootstrap 5 grid (CDN), Bootstrap Icons (CDN), SQLite (configured but unused), optional zxcvbn.

## Architecture
Browser: `app.js` (analysis + generator). Server: Django views render templates; `SecurityHeadersMiddleware` sets CSP (`connect-src 'none'`), `no-store`, and related headers. No models, sessions, auth or cookies.

## Structure
```
securepass_analyzer/
  manage.py  requirements.txt  README.md
  config/        settings.py urls.py wsgi.py asgi.py
  analyzer/      views.py urls.py middleware.py tests.py apps.py
    templates/analyzer/   base home analyzer generator tips about
    static/analyzer/      css/style.css  js/app.js  js/vendor/  images/shield.svg
```

## Scoring methodology
1. Pool size from character types present (26 / 26 / 10 / 33).
2. Effective length = length minus repeated characters/chunks, sequence and keyboard-walk characters, and matched personal-detail characters.
3. Entropy estimate = effective length × log2(pool).
4. Common passwords (including simple leetspeak like `p@ssw0rd`) cap entropy at 10 bits.
5. If `zxcvbn.js` is present, entropy = the lower of the rule estimate and zxcvbn's guess estimate.
6. Score = entropy in bits, capped at 100; passwords under 8 characters are capped at 19.
7. Categories: 0–19 Very Weak, 20–39 Weak, 40–59 Fair, 60–79 Strong, 80–100 Very Strong.

Guessing time = 2^(entropy−1) ÷ 10^10 guesses/second (fast-hash offline attack). It is an estimate, not a guarantee. Without zxcvbn, the dictionary is small, so a long passphrase of ordinary words can be overrated; add zxcvbn for better results.

## Privacy and security
No password endpoint (POST returns 405), no database models, no cookies, no storage APIs, no logging of input, CSP blocks page-script network access, `Referrer-Policy: no-referrer`, DEBUG and SECRET_KEY come from environment variables (set `DJANGO_DEBUG=0` and `DJANGO_SECRET_KEY` for production). Do not enter real passwords on untrusted sites.

## Installation (Windows Command Prompt)
```
mkdir securepass_analyzer_project
cd securepass_analyzer_project
:: copy the project files here so manage.py is in this folder
python -m venv venv
venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements.txt
python manage.py runserver
```
Open http://127.0.0.1:8000/. No migrations are needed. Optional zxcvbn: copy `dist/zxcvbn.js` from the npm package `zxcvbn` into `analyzer\static\analyzer\js\vendor\zxcvbn.js`.

Troubleshooting: `python` not found → install Python and tick "Add to PATH" or use `py -m venv venv`; activation blocked → use Command Prompt, not PowerShell; port busy → `python manage.py runserver 8001`; styles missing offline → Bootstrap/Icons load from a CDN, so connect once or self-host them.

## Usage
Open Analyzer, type a password (optionally add personal details), read the score, checklist and advice. Open Generator to create a password and copy it.

## Testing
`python manage.py test` runs page, header, privacy and source-scan tests. Browser console checks: `SPA.analyze("password").label`, `SPA.categorize(59)` (Fair), `SPA.categorize(60)` (Strong), `SPA.generate(20, ["digit"])`. Manual UI checks: toggle show/hide, clear, resize to 375px width, tab through all controls, generate with every box unchecked (error appears), copy.

## Limitations
Estimates only; no breach database; small built-in dictionary; clipboard needs HTTPS or localhost; CDN assets need internet.

## Future enhancements
Bundled wordlist, offline Bootstrap, passphrase generator, Selenium/Playwright tests, local k-anonymity breach check (opt-in).
