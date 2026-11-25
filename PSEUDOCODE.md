# Wealthness — Feature Pseudocode and Page Connections

This document explains the main features in simple pseudocode and shows how they share information. It is written to match the current implementation in `script.js` and the existing HTML pages.

## 1. How the website is connected

Wealthness uses one shared JavaScript data object. The object is loaded from the browser's Local Storage when a page opens. When a user adds, edits, or removes a record, the app saves the object and refreshes the relevant page sections.

```text
User action on any page
        |
        v
HTML form or button
        |
        v
JavaScript event handler
        |
        v
Validate and update shared data object
        |
        v
Save data to browser Local Storage
        |
        v
Refresh page sections that use the changed data
        |
        v
Dashboard, Budget, Savings, Debt, Calendar,
and Financial Health show the updated records
```

**Important:** Local Storage belongs to the current browser on the current device. The project does not connect to a real bank, GCash, or Maya account, and data is not automatically synchronized between devices.

## 2. Shared data

| Data field | Used by |
|---|---|
| `transactions` | Dashboard, Budget & Expenses, Financial Insights, Financial Health, Calendar |
| `budgets` | Budget & Expenses and Financial Health |
| `goals` | Savings Goals, Dashboard, Financial Health |
| `debts` | Debt Management, Savings Planner, Financial Health, Calendar |
| `recurring` | Budget & Expenses and Financial Calendar |
| `plannedPayments` | Financial Calendar |
| `challengeState` | Savings Challenges section in Savings Goals |
| `planner` | Savings Planner and Dashboard savings suggestion |
| `settings` | Settings and saved profile preferences |
| `customCategories` | Transaction forms, recurring entries, planned payments, and budget limits |

## 3. Feature pseudocode

### A. Dashboard / Overview

**Purpose:** Show a summary of the user's recorded finances.

```text
START Dashboard
    Load shared financial data
    Calculate all-time income and expenses
    Calculate this month's income and expenses
    Calculate available balance
    Add saved amounts from all savings goals
    Display the summary cards
    Group this month's expenses by category
    Display the cash-flow chart and recent transactions
    Display savings progress and financial health summary
END Dashboard
```

**Connections:** Reads transactions from Budget & Expenses, goals from Savings Goals, debts from Debt Management, and budget limits from Budget & Expenses.

### B. Budget & Expenses

**Purpose:** Record and organize income and expenses.

```text
WHEN user submits a transaction
    Read date, type, description, category, and amount
    Check that required values and amount are valid
    Add or update the transaction in shared data
    Save shared data to Local Storage
    Refresh the dashboard, budget list, insights, and health summary
END
```

**Connections:** Transactions affect the Dashboard, Financial Insights, Financial Health, and cash-flow chart. Budget limits are also used by the Financial Health estimate.

### C. Quick Budget Calculator

**Purpose:** Estimate a savings amount from monthly income.

```text
START Budget Calculator
    Read monthly income and savings percentage
    Check that percentage is between 0 and 100
    Calculate suggested savings = income x percentage / 100
    Calculate remaining amount = income - suggested savings
    Display both estimates
END
```

**Connections:** It is located inside Budget & Expenses. It only displays an estimate and does not change saved transactions or budgets.

### D. Savings Goals

**Purpose:** Create goals and track contributions.

```text
WHEN user creates a goal
    Read goal name and target amount
    Check that the name is not empty and target is greater than zero
    Add a goal with saved amount set to zero
    Save shared data
    Refresh savings goals and dashboard summaries
END

WHEN user adds a contribution
    Find the selected goal
    Add the contribution to its saved amount
    Save shared data
    Refresh savings progress and dashboard summaries
END
```

**Connections:** Goal progress appears in the Dashboard and Financial Health summary.

### E. Savings Goal Calculator

**Purpose:** Estimate the monthly amount needed to reach a target.

```text
START Savings Goal Calculator
    Read target, amount already saved, and number of months
    Check that target and months are valid
    Remaining amount = maximum of zero and (target - already saved)
    Monthly estimate = remaining amount / months
    Display the monthly estimate
END
```

**Connections:** Located in Savings Goals. The calculation does not automatically add money to a goal.

### F. Emergency Fund Calculator

**Purpose:** Estimate an emergency fund target.

```text
START Emergency Fund Calculator
    Read essential monthly expenses
    Read number of months to cover
    Check the entered expense amount
    Target = essential monthly expenses x selected months
    Display the target
END
```

**Connections:** Located in Savings Goals. The Savings Planner also shows three- and six-month estimates using its essential-expense value.

### G. Savings Challenges

**Purpose:** Help users track optional daily check-ins.

```text
WHEN user starts a challenge
    Save selected duration (7 or 30 days)
    Start with an empty list of check-in dates
    Display challenge progress

WHEN user checks in
    Get today's local date
    If today's date is not already saved
        Add today's date to the check-in list
        Save shared data
    Otherwise
        Tell the user that today's check-in is already recorded
    Update progress display
END
```

**Connections:** Challenge progress is stored in shared data and shown in Savings Goals. A check-in is a habit tracker; it does not automatically change a savings goal balance.

### H. Debt Management

**Purpose:** Record debts and monitor repayments.

```text
WHEN user records a debt
    Read debt name, original amount, and due date
    Check that the name and amount are valid
    Set remaining balance equal to original amount
    Save shared data
    Refresh debt list, planner, health summary, and calendar
END

WHEN user records a payment
    Find the selected debt
    Subtract payment from remaining balance
    Do not allow remaining balance to go below zero
    Save shared data
    Refresh connected summaries
END
```

**Connections:** Debt balances appear in Debt Management, the Savings Planner, Financial Health, and due dates appear in Financial Calendar.

### I. Financial Calendar

**Purpose:** Bring planned and upcoming financial events together.

```text
START Financial Calendar
    Read planned payments
    Read debts that have a due date and remaining balance
    Read recurring entries with a next scheduled date
    Combine the events into one list
    Sort events by date
    Mark each event as overdue, due today, or upcoming
    Display the schedule and totals
END
```

**Connections:** Reads planned payments from the Calendar form, due dates from Debt Management, and next dates from recurring transactions in Budget & Expenses.

### J. Financial Health Overview

**Purpose:** Give an educational estimate based on the user's saved records.

```text
START Financial Health Overview
    Read this month's transactions
    Read budget limits, savings goals, and debts
    Calculate income, expenses, savings progress, and remaining debt
    Calculate the simple estimate score
    Create suggestions based on missing or useful records
    Display the score, summaries, and suggestions
END
```

**Connections:** Reads data from Budget & Expenses, Savings Goals, and Debt Management. The score is only a simple project estimate, not a professional financial assessment.

### K. Financial Guides

**Purpose:** Provide learning material about budgeting, saving, needs and wants, emergency funds, and debt.

```text
START Financial Guides
    Display available guide topics
    User opens a topic
    Show the explanation for that topic
END
```

**Connections:** Guides are educational content. They do not change financial records.

### L. Settings and Backups

**Purpose:** Manage display preferences and protect user records.

```text
WHEN user changes theme
    Switch light or dark appearance
    Save theme preference in Local Storage

WHEN user exports a backup
    Convert shared data into JSON
    Download the JSON file

WHEN user restores a backup
    Read selected JSON file
    Check that it contains transaction data
    Ask user to confirm replacement
    Replace shared data and save it
    Refresh connected page sections
END
```

**Connections:** Settings controls the shared appearance preference and the backup/import process for the shared financial data.

## 4. How a transaction updates other pages

Example: a user records a ₱500 food expense in Budget & Expenses.

1. The transaction form checks the entered information.
2. JavaScript adds the expense to `data.transactions`.
3. `save()` stores the updated data in Local Storage.
4. `refresh()` redraws the page sections.
5. The Dashboard recalculates monthly spending and the cash-flow chart.
6. Financial Insights groups the expense under Food.
7. Financial Health recalculates its summary and suggestions.

## 5. Main JavaScript functions

| Function | Responsibility |
|---|---|
| `loadData()` | Loads saved records and applies default values |
| `save()` | Saves the shared data object to Local Storage |
| `monthTransactions()` | Filters transactions for the current local month |
| `totals()` | Calculates income and expense totals |
| `renderOverview()` | Displays Dashboard information |
| `renderChart()` | Builds the six-month cash-flow chart |
| `renderBudget()` | Displays and filters transactions and budgets |
| `renderSavings()` | Displays savings goals |
| `updatePlanner()` | Calculates savings and emergency fund estimates |
| `renderDebt()` | Displays debt balances and repayment progress |
| `renderCalendar()` | Combines and displays scheduled events |
| `renderHealth()` | Displays the educational financial health estimate |
| `refresh()` | Re-renders connected sections after data changes |
| `initPage()` | Sets up page events and starts the initial rendering |

## 6. Notes for testing

- Add an income or expense, then check the Dashboard and Financial Health overview.
- Add a savings goal or contribution, then check the Dashboard savings progress.
- Add a debt with a due date, then check Financial Calendar.
- Add a recurring transaction and verify its next scheduled date.
- Start a savings challenge, check in, refresh the page, and confirm that progress remains.
- Export a backup, then test restoring it using a copy of the file.
- Test dates around the beginning of a month to confirm that chart labels use the local calendar month.
