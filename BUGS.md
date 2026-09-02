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

**How to reproduce:** Look at the Balances panel. Diya paid the Uber to airport ($60) but only Aisha and Ben were on the split. Diya should get the full $60 back, but her balance is wrong — she shows as owing money instead of being owed.

**What is wrong:** In `src/lib/balances.js` there was an extra block that ran when the payer was not included in the split. It incorrectly subtracted a share from the payer's balance, reducing their credit. According to the spec, if someone pays but is not on the split, they should get the full amount back — nothing should be deducted from them.

**What I changed:** Removed the incorrect `if` block (lines 16-19) from `computeBalances` in `src/lib/balances.js`. The shares loop above it already correctly assigns costs only to the people in `splitWith`, so the payer's credit was already right — the extra subtraction was the bug.

---