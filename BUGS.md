# Bugs found

Add one section per issue. Bug 1 is filled in to show the format - fix it, then write what you changed. Copy the blank template for the rest.

Keep this file in the repo and **commit it** with your fixes.

---

## Bug 1

**How to reproduce:** Open the app. The expense list says "Newest first". The first row is Wine (7 Mar). Board game (15 Mar) is further down.

**What is wrong:** The list is showing oldest expenses first. Newest should be at the top.

**What I changed:** In `src/components/ExpenseList.jsx` line 62, the sort comparator was `dateValue(a.date) - dateValue(b.date)` (ascending - oldest first). Swapped it to `dateValue(b.date) - dateValue(a.date)` (descending - newest first) so Board game (15 Mar) now appears at the top and Wine (7 Mar) at the bottom.

---

## Bug 2

**How to reproduce:** Look at the Balances panel. Diya paid the Uber to airport ($60) but only Aisha and Ben were on the split. Diya should get the full $60 back, but her balance is wrong - she shows as owing money instead of being owed.

**What is wrong:** In `src/lib/balances.js` there was an extra block that ran when the payer was not included in the split. It incorrectly subtracted a share from the payer's balance, reducing their credit. According to the spec, if someone pays but is not on the split, they should get the full amount back - nothing should be deducted from them.

**What I changed:** Removed the incorrect `if` block (lines 16-19) from `computeBalances` in `src/lib/balances.js`. The shares loop above it already correctly assigns costs only to the people in `splitWith`, so the payer's credit was already right - the extra subtraction was the bug.

---

## Bug 3

**How to reproduce:** Add an expense of $100 split equally between 3 people. Check the balances - the numbers do not add up to exactly $100. Each person shows $33.33, which totals $99.99, not $100.

**What is wrong:** In `src/lib/money.js`, the `splitEqual` function rounds each person's share independently to 2 decimal places and gives everyone the same amount. This loses or gains a cent when the amount doesn't divide evenly. The README says shares must cover the full bill exactly.

**What I changed:** Updated `splitEqual` in `src/lib/money.js` to give all but the last person the base rounded share, then give the last person whatever is left over (`amount - assigned`). This way the shares always sum to exactly the original amount, no matter how many people are splitting.

---

## Bug 4

**How to reproduce:** Filter the expense list by a category (e.g. "Food"). Then click Delete on one of the visible expenses. The wrong expense gets deleted - the one that was at that position in the unfiltered list, not the one you clicked.

**What is wrong:** `ExpenseList.jsx` was using the array index from the filtered and sorted list to identify which expense to delete or edit. But the reducer in `store.js` was using that same index to splice `state.expenses`, which is the full unfiltered array. When filters are active the indexes don't match, so the wrong expense gets removed or updated.

**What I changed:** Changed `ExpenseList.jsx` to pass `expense.id` instead of the array index to `onDeleteAt` and `onUpdateAt`. Updated `App.jsx` to forward that `id` in the dispatch action. Changed the `DELETE_EXPENSE` and `UPDATE_EXPENSE` cases in `src/state/store.js` to find the expense by `id` using `.filter()` and `.map()` instead of using an index to splice.

---

## Bug 5

**How to reproduce:** Open the app, add a new expense, then refresh the page. The expense list order becomes wrong after refresh even though it was correct before.

**What is wrong:** In `src/state/store.js`, the `loadState` function calls `hydrate()` when loading from the seed data (which converts date strings into proper `Date` objects), but when loading from `localStorage` it just calls `JSON.parse()` directly and skips hydration. JSON cannot store `Date` objects, so after a refresh all dates come back as plain strings. The sort in `ExpenseList` then compares strings instead of dates and produces incorrect ordering.

**What I changed:** In `src/state/store.js`, changed `return JSON.parse(raw)` to `return hydrate(JSON.parse(raw))` so dates are always converted to `Date` objects regardless of where the data is loaded from.

---

## Bug 6

**How to reproduce:** Open the Balances panel. Aisha paid for Groceries ($100) split 3 ways. Her balance shows "owes" when she should show "is owed" since she put money in. Carlos, who owes money, shows "is owed" instead of "owes".

**What is wrong:** The balance value is calculated as paid minus consumed. A positive balance means the person paid more than their share and the group owes them. A negative balance means they consumed more than they paid and they owe the group. The labels in `BalancesPanel.jsx` had this logic backwards - positive showed "owes" and negative showed "is owed".

**What I changed:** Swapped the label text and CSS class names in `src/components/BalancesPanel.jsx` so that a positive balance shows "is owed" (with class `owed`) and a negative balance shows "owes" (with class `owe`).

---