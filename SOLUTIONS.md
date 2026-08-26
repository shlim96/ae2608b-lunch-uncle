# Seeded bugs (facilitator copy — remove before publishing)

| # | Symptom | Location | Fix |
|---|---------|----------|-----|
| 1 | Uncle always answers "Just go Berseh Food Centre lah." for any message mentioning food | `src/loop.js`, the guard at the top of `runLoop` | The fallback was meant for "no Places key AND the user asked about food". The `||` should be `&&`: `if (!env.GOOGLE_PLACES_API_KEY && FOOD_WORDS.test(message))`. Better still, drop the regex and only fall back when the tool actually fails. |
| 2 | All recommendations are in Bedok, and distances are measured from Bedok | `src/tools.js`, `findLunchPlaces`: `const centre = { latitude: 1.3236, longitude: 103.9273 }` | Replace the inline literal with the `CT_HUB_2` constant declared at the top of the file. It is currently unused. |
| 3 | "Compare every restaurant within 2km" never returns; the loop runs until the platform kills the request | `src/loop.js`, the `while (round < MAX_ROUNDS)` loop | `round` is never incremented, so the cap is never reached. Add `round++` at the end of the loop body (or use a `for` loop). |
| 4 | Uncle recommends places that are closed | `src/tools.js`, `formatPlaces` | The FieldMask asks for `currentOpeningHours` and the API returns it, but `formatPlaces` destructures only `displayName, rating, location`. Add `currentOpeningHours` and emit `open_now: currentOpeningHours?.openNow ?? null`. |
| 5 | Long conversations get slower and cost more every turn | `src/loop.js`, the `messages` array built from `...history` | History is never truncated. Keep the last N turns, e.g. `...history.slice(-20)`. |
| 6 | Correct behaviour, but prompt caching never hits so every call pays full input price | `src/prompt.js`, `buildSystemPrompt` | The prompt begins with a fresh UUID and timestamp, so the cached prefix changes every request. Remove the `Request ... at ...` header (put request IDs in logs, not prompts). If the time is needed, append it at the end of the prompt or in the user message. |

## Verifying

- Bug 1: ask anything containing "eat", "lunch", "food", "makan", "hungry", "restaurant" or "hawker".
- Bugs 2 and 4: temporarily change `||` to `&&` in the guard, then ask "any Japanese food?" Results will be in Bedok and some will be closed.
- Bug 3: with the guard bypassed, ask "compare every restaurant within 2km" and watch `wrangler dev` logs — the `round N:` counter stays at 0.
