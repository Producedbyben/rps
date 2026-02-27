# Code Review Findings

## 1) URL `seed` parsing is inconsistent with the input validation path

**Severity:** Medium

`readInitialSettingsFromUrl()` parses `seed` using `parseInt` and returns it directly as manual input, but this path does not enforce the same digit-only and range handling used by `resetGame()`. This means values like `seed=-1`, `seed=123abc`, or very large values are accepted from URL query params and then silently coerced by `initFromSeed(seed >>> 0)`, producing wrapped or truncated seeds.

### Why this matters
- The seeded run UX implies reproducibility and normalized unsigned 32-bit behavior.
- URL-based seeds currently do not follow the same normalization contract as the seed input box.
- Shared challenge links can initialize with surprising seeds when query params are malformed.

### Evidence
- URL path uses `parseInt` only. (`readInitialSettingsFromUrl`)
- Input path uses regex + BigInt range clamp. (`resetGame`)
- Initialization coerces with `>>> 0`, causing wraparound. (`initFromSeed`)

### Suggested fix
- Reuse the same validation/normalization logic for both URL parsing and manual input.
- Reject non-digit URL seeds (`^\d+$`) and clamp with `BigInt` to `[0, 4294967295]` for consistency.

## 2) Query parsing accepts partially numeric values (`parseInt`) as valid seeds

**Severity:** Low

`parseInt` accepts numeric prefixes, so `?seed=42xyz` is treated as seed `42` instead of invalid input. This is likely not intended for a deterministic challenge-link experience.

### Suggested fix
- Replace `parseInt` acceptance with a strict full-string numeric check before conversion.
