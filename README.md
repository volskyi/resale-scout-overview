# Resale Scout: overview

> **Source code is private.** This repository describes the system's logic and architecture.

**An autonomous agent that watches second-hand classifieds marketplaces 24/7, finds audio equipment
listed below its real market value, and pushes the deals to Telegram with photos, price, distance and
estimated margin.**

![Python](https://img.shields.io/badge/Python-3-3776AB)
![Playwright](https://img.shields.io/badge/Playwright-browser-2EAD33)
![LLM](https://img.shields.io/badge/LLM-vision%20%2B%20tiered-blueviolet)
![SQLite](https://img.shields.io/badge/SQLite-state-003B57)
![Telegram](https://img.shields.io/badge/Telegram-bot-26A5E4)

## The idea

The best resale deals aren't the ones everyone sees. They're the listings with a misspelled model name,
a vague title like "black speaker", or a "not working" tag on something that needs one cheap part.
Keyword alerts miss all of them. This system is built to find exactly those.

## How it decides a listing is worth it

1. **Wide sweep.** Dozens of queries, including generic words and the typical misspellings sellers make
   with brand and model names, where competition is lowest
2. **Model recognition from photos.** A vision model identifies what the item really is when the title
   is wrong or empty
3. **Below-market check.** Estimates market value from the median of comparable real listings and flags
   significantly cheaper ones
4. **Broken + part logic.** For "not working / for parts" listings, it infers the likely failed component,
   prices the replacement, and flags the lot only if *price + part* is still at least a set margin below market
5. **Status of the model.** Separates collectible vintage from low-value mass-market gear that's cheap
   for a reason
6. **True buyer cost.** Margin is computed from what the buyer actually pays (price, platform
   buyer-protection fee, shipping), not from the sticker price. Pickup has no fees
7. **Distance tiers.** Nearby lots alert normally; distant ones only when the margin justifies the trip
   ("🚗 worth a trip")
8. **No repeats.** A deep baseline scan on first run, then only new listings (SQLite state)

## Architecture

```
 scheduler ─► marketplace adapters (Playwright / HTTP, polite pacing)
                    │
              filters & fee model ──► candidate shortlist
                    │
              tiered AI analysis: cheap model first, stronger model on the shortlist
              (key pools, rate limiting, per-run budget cap)
                    │
              SQLite (seen listings, price research) ──► Telegram bot (photos, margin, actions)
```

- Runs 24/7 as a supervised service on a home server
- Pluggable marketplace adapters behind one interface
- Configurable thresholds (radius, minimum margin, AI budget) in YAML
- The cost model was validated against real purchases to the cent

## My role

Defined the deal criteria from real resale cases, designed the scoring logic and AI tiering, built and
operate the service.
