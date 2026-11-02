# Portfolio Rebalance Automation Assessment

Java implementation of the rebalancing exercise, with Cucumber BDD tests run through TestNG.

The assignment asks: given account **ABC** ($100,000, 100% vested) and five securities, produce the number of shares to **buy** or **sell** so each holding reaches its target weight. Tests must confirm that share counts and actions are correct. Assumptions are recorded in [ASSUMPTIONS.md](ASSUMPTIONS.md).

## Overview

The application under test is a small in-process service, not a UI or HTTP API. Automated tests call `RebalancingService` directly.

| Layer | Component | Role |
| --- | --- | --- |
| Domain | `Account` | Named account, total asset value, list of holdings |
| Domain | `Security` | Ticker, target %, current %, unit price |
| Domain | `RebalanceResult` | Ticker, action (`BUY` / `SELL` / `NONE`), whole-share quantity |
| Logic | `RebalancingService.calculate` | One-security share math and field validation |
| Logic | `RebalancingService.rebalance` | Walk all holdings, enforce a 100% target mix, return trades |
| BDD | `src/test/resources/features/rebalance.feature` | Scenarios for happy path, rounding, validation, and account-level rules |
| Glue | `RebalanceSteps.java` | Cucumber step definitions |
| Runner | `TestRunner` + `testng.xml` | TestNG suite that loads the feature files |

**Share formula** (see [ASSUMPTIONS.md](ASSUMPTIONS.md) for rounding and edge cases):

```
targetValue  = totalAsset × target% / 100
currentValue = totalAsset × current% / 100
difference   = targetValue − currentValue
action       = BUY if difference > 0, SELL if difference < 0, NONE if 0
shares       = round(|difference| / unitPrice)   // Java Math.round, int shares
```

Assignment sample for account ABC (`$100,000`):

| Security | Target % | Current % | Price | Action | Shares |
| --- | --- | --- | --- | --- | --- |
| IBM | 20 | 10 | 150 | BUY | 67 |
| MSFT | 20 | 20 | 90 | NONE | 0 |
| ORCL | 20 | 30 | 220 | SELL | 45 |
| AAPL | 20 | 20 | 450 | NONE | 0 |
| HD | 20 | 20 | 70 | NONE | 0 |

Manual cases mapped to the feature file live in `docs/Manual_Test_Cases_Feature_Mapping.xlsx`.

## Prerequisites

| Tool | Version | Notes |
| --- | --- | --- |
| JDK | **17 or newer** | `pom.xml` compiles with `--release 17`. JDK 21 (for example Eclipse Temurin 21) is fine. |
| Apache Maven | 3.8+ | Uses compiler plugin 3.13.0 and Surefire 3.2.5. |
| OS | Windows / macOS / Linux | Windows PowerShell notes below. |


## Running the Tests

Run from IntelliJ: Execute TestRunner.java
Or using Maven:mvn test

## Reports
target/cucumber-report.html

## Test Coverage
The framework validates:

Buy calculations
Sell calculations
No-action scenarios
Rounding logic
Invalid inputs
Account-level validations
Portfolio rebalancing rules
