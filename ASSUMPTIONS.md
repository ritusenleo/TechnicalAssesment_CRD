# Assumptions

Problem statement: account ABC, $100,000 total assets, 100% vested, holdings IBM / MSFT / ORCL / AAPL / HD with the given target %, current %, target variance, and unit price. Required output is shares to buy or sell per security.

---


## 1. Business logic (formulas and actions)

### 1.1 Per-security calculation (`calculate`)

```
targetValue  = totalAsset * targetPercent / 100.0
currentValue = totalAsset * currentPercent / 100.0
difference   = targetValue - currentValue
```

| `difference` | `action` | `shares` |
| --- | --- | --- |
| `> 0` | `BUY` | `round(\|difference\| / price)` |
| `< 0` | `SELL` | `round(\|difference\| / price)` |
| `== 0` | `NONE` | `0` (also `round(0)` ) |

Direction matches the assignment’s variance note (negative variance / underweight ⇒ buy).

Assignment ABC numbers:

- IBM: `$10,000` / `$150` = `66.666…` → **BUY 67**
- MSFT / AAPL / HD: difference `0` → **NONE 0**
- ORCL: `$10,000` / `$220` = `45.4545…` → **SELL 45**

### 1.2 Account calculation (`rebalance`)

1. Reject a null account (`Account is required`).
2. For each holding: reject missing/blank ticker (`Security name is required`); add `targetPercentage` to a running total; call `calculate` with the account’s `totalAsset`.
3. If there is at least one holding and `|sum(target%) − 100| > 0.001`, reject (`Target allocation must total 100`).
4. Return one `RebalanceResult` per holding, **same order** as the input list.

`calculate` is reused so account tests and single-name tests share one formula.

### 1.3 Validation

Order inside `calculate` (first failure wins):

1. `totalAsset <= 0` → `Asset value must be greater than 0` (covers `0` and negatives).
2. `price <= 0` → `Price must be greater than 0`.
3. `targetPercent < 0` → `Target percentage cannot be negative`.
4. `targetPercent > 100` → `Target percentage cannot exceed 100`.

Account extras:

- Null/blank security name (Java `isBlank()`, so whitespace-only names fail).
- Sum of targets must be 100% ± `0.001` when holdings exist.


## 2. Rounding and numeric behavior

| Rule | Detail |
| --- | --- |
| Type | `double` for money and percents; `int` for shares. |
| Share rounding | `(int) Math.round(Math.abs(difference / price))`. |
| Half-up | Java `Math.round(double)` is **floor(x + 0.5)** for finite values, so **2.5 → 3**, **3.5 → 4**. Feature example: `$10,000 / $4,000 = 2.5` → **3** shares (buy or sell). |
| Direction does not change rounding | Absolute value is taken **before** rounding, so buy and sell of the same notional/price get the same share count. |
| Action vs rounding | Action uses the **unrounded** `difference`. A tiny overweight still SELLs even if shares round to 0. |
| `NONE` | Only exact `difference == 0.0`. No epsilon on the dollar difference. Percents that *should* match can still trade if binary floats disagree (not seen with the assignment’s 10/20/30 values). |
| Allocation total | Compared with absolute epsilon **0.001**, not exact `== 100`. Allows `33.33 + 33.33 + 33.34`. |
| No currency scale | Values are not quantized to cents before dividing by price. |

---



## 3. Technical

| Assumption | Detail |
| --- | --- |
| Language / build | Java 17 bytecode (`maven.compiler.release` = 17), Maven, TestNG 7.10.2, Cucumber 7.18.1. |
| Runtime JDK | **17+**. JDK 21 is acceptable. No `--enable-preview`, no `module-info.java`. |
| Windows terminals | User-level `JAVA_HOME` may be unset in the current PowerShell until it is copied from User/Machine env (see README). |
| Cucumber feature path | `src/test/resources/features` as a **source** path in `@CucumberOptions`. Run Maven from the **module root**. |
| Logging | `slf4j-simple` so Cucumber/TestNG do not warn about a missing binding. |

---

