# Green & Tasty — Restaurant AQA Framework

End-to-end and API test automation suite for the Green & Tasty restaurant management platform.

Developed as the final capstone project for **IT Marathon 2026 (Task #10 — Test Automation)**.

---

## 📌 Testing Scope & Approach

* **Direction:** **Direction 1 — Group Pre-Order** (testing full custom backend and frontend implementation).
* **Architecture Pattern:** **BDD (Gherkin/Reqnroll) + Screenplay Pattern + TUnit + Playwright for .NET**.
* **Framework Version:** `.NET 10.0` with Microsoft Testing Platform.
* **CI/CD Quality Gates:** GitHub Actions (`aqa-quality.yml`, `aqa-tests.yml`).

---

## 🎯 Test Levels & Covered Scenarios

### 🟢 Level 1: Viewing Group Pre-Order Participants
* **Owner overview:** Reservation owner sees all joined participants, their chosen items, individual item quantities, line totals, and group total sum.
* **Unconfirmed carts:** A guest who is still choosing dishes and hasn't submitted their personal cart is clearly marked with `still choosing` status.
* **Empty group state:** A group with no joined participants displays an empty participant list (`[]`) and zero total (`$0.00`).
* **Security & Authorization:** Group session details can only be viewed by the reservation owner. Unauthorized requests or requests from other customers/waiters are rejected (`401/403`).
* **Non-existent group:** Requests to non-existent group IDs return `404 Not Found`.

### 🔵 Level 2: Consolidated Pre-Order Confirmation
* **Consolidation logic:** The owner consolidates and submits the order. Only confirmed guest carts are aggregated; duplicate items ordered by different guests are combined with summed quantities.
* **Status transitions:**
  * Associated restaurant order status updates to `SubmittedByCustomer`.
  * Group reservation status updates to `Submitted`.
* **Waiter visibility:** The assigned location waiter can view the consolidated pre-order composition matching the owner's confirmation.
* **Validation & Constraints:** Pre-order confirmation is rejected under invalid conditions:
  * No confirmed guest carts.
  * Confirmed carts contain no items.
  * Attempting to re-submit an already submitted group reservation (`409 Conflict` / `400 Bad Request`).
* **Non-owner submission:** Attempts to confirm the pre-order by a user other than the owner are forbidden (`403 Forbidden`).

### 🟣 Level 3: Guest Removal from Group
* **Participant removal:** Owner can remove a guest session from an active group. The participant is removed and the grand total is recalculated dynamically.
* **Submitted participant removal:** Removing a guest who had already submitted their cart successfully removes them and excludes their items from future order consolidation.
* **Cross-group isolation:** Removing a guest belonging to another group reservation returns `400 Bad Request`.
* **Non-existent participant:** Removing a non-existent guest session returns `404 Not Found`.
* **Non-owner removal:** Removing guests by unauthorized users or other customers is refused (`401/403`).
* **End-to-End Smoke Flow:** Full end-to-end lifecycle: multiple guests join and select dishes -> owner inspects -> removes one guest -> owner submits consolidated pre-order -> waiter inspects the order matching only active guests.

---

## 🛠 Handled Undefined Specifications & Edge Cases

As per marathon assignment guidelines:
1. **Empty group reservation:** Returns `participants: []` and `total: 0`.
2. **Re-submitting confirmed group:** Returns `409 Conflict` (or `400 Bad Request`) with error details.
3. **Non-owner actions:** Attempting to view or modify group sessions without owner credentials returns `401 Unauthorized` or `403 Forbidden`.
4. **Invalid cross-group removal:** Returns `400 Bad Request`.

---

## 🚀 Running the Tests Locally

### Prerequisites
* [.NET 10.0 SDK](https://dotnet.microsoft.com/download)
* Node.js (v20+ recommended for Gherkin linting)

### Configuration
Copy `.env.example` to `.env` and configure your environment settings:
```bash
cd aqa/restaurant-aqa-framework
cp .env.example .env
```
> ⚠️ **Note:** `.env` is git-ignored and must never be committed.

### Build and Verification
```bash
# Verify formatting against .editorconfig
dotnet format restaurant-aqa-framework.csproj --verify-no-changes

# Build test project (treats analyzer warnings as errors)
dotnet build
```

### Running Test Scenarios
```bash
# Discover all available test scenarios
dotnet run --no-build -- --list-tests

# Run all test scenarios with TRX reporting
dotnet run --no-build -- --report-trx

# Run only Direction 1 Group Pre-Order scenarios
dotnet run --no-build -- --treenode-filter "/*/*/*/*[Category=group-preorder]"

# Run only smoke scenarios
dotnet run --no-build -- --treenode-filter "/*/*/*/*[Category=smoke]"
```

---

## 📊 Test Reports & Artifacts

All test runs generate artifacts inside `aqa/restaurant-aqa-framework/TestResults/`:
* **TUnit HTML Report:** `TestResults/*-report.html` — interactive dashboard of test runs.
* **TRX Report:** `TestResults/*.trx` — standard Visual Studio / CI test results XML.
* **Reqnroll Report:** `TestResults/reqnroll/` — BDD feature-level execution summaries.
* **Playwright Traces:** `TestResults/traces/*.zip` — post-mortem traces for failed UI tests (open with `npx playwright show-trace <file>.zip`).

---

## 🌐 CI/CD Quality Gates & GitHub Actions

The repository includes two GitHub Actions workflows:

1. **`aqa-quality.yml` (Quality Gate):**
   * Runs on every push and pull request to `main`.
   * Verifies no generated or binary files (`bin/`, `obj/`, `TestResults/`, `.env`) are tracked.
   * Enforces `dotnet format --verify-no-changes`.
   * Enforces strict static analysis and builds the solution.
   * Runs Gherkin linting and test discovery.

2. **`aqa-tests.yml` (Test Execution Pipeline):**
   * Runs on schedule (nightly cron) and manual execution (`workflow_dispatch`).
   * Accepts filters (`all`, `ui`, `api`, or custom tree filters).
   * Injects secrets and repository variables for target environments.
   * Publishes TRX, HTML, and Playwright trace artifacts.
   * Generates a GitHub Job Summary table.
