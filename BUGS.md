# Bugs found

Add one section per issue. Bug 1 is filled in to show the format - fix it, then write what you changed. Copy the blank template for the rest.

Keep this file in the repo and **commit it** with your fixes.

---

## Bug 1

**How to reproduce:** Open the app. The expense list has a "Newest first" label at the top. But Wine (7 Mar) is showing first and Board game (15 Mar) is buried lower down. That's backwards.

**What is wrong:** The sort was doing oldest-first. The comparator was `dateValue(a.date) - dateValue(b.date)` which puts smaller (older) dates at the top. The label says newest first, so it's literally doing the opposite of what it claims.

**What I changed:** Flipped the comparator in `src/components/ExpenseList.jsx` to `dateValue(b.date) - dateValue(a.date)`. Now newer dates bubble to the top and it matches the label.

---

## Bug 2

**How to reproduce:** Check the Balances panel. Diya paid $60 for the Uber to airport — only Aisha and Ben were in the car. Diya shouldn't owe herself anything. Her balance should fully reflect that $60 she fronted, but it doesn't.

**What is wrong:** There were a few extra lines in `src/lib/balances.js` inside `computeBalances` that were subtracting a share from the payer even when the payer wasn't in the split at all. So Diya was silently losing part of her own credit for no reason.

**What I changed:** Deleted those 4 lines. The loop above already handles charging only the people listed in `splitWith`, so those extra lines were just wrong. Removing them gives the payer their full credit back.

---

## Bug 3

**How to reproduce:** Add a $100 expense and split it equally between 3 people. Check the shares — you'll see $33.33 + $33.33 + $33.33 = $99.99. A cent just vanishes.

**What is wrong:** `splitEqual` in `src/lib/money.js` was rounding every person's share the same way. When you can't divide evenly, that rounding eats a cent. The README is clear that shares must always add up to exactly the original total.

**What I changed:** Reworked `splitEqual` so everyone except the last person gets the base rounded share. The last person gets whatever's left over from the total. That way the shares always add up exactly, no matter what.

---

## Bug 4

**How to reproduce:** Filter the list by a category like "Food". Hit Delete on one of the visible expenses. The wrong expense disappears — not the one you clicked.

**What is wrong:** The delete was passing the row's index from the *filtered* list to the reducer. But the reducer was using that index against the *full* unfiltered list. Those indexes don't line up when a filter is active, so you end up deleting a completely different expense.

**What I changed:** Switched `ExpenseList.jsx` to pass `expense.id` instead of the index. Updated `App.jsx` to send that id in the dispatch. Changed the `DELETE_EXPENSE` and `UPDATE_EXPENSE` cases in `store.js` to look up expenses by id rather than by position.

---

## Bug 5

**How to reproduce:** Open the app — looks fine. Hit refresh. Sort order is now broken.

**What is wrong:** On first load, `hydrate()` runs and turns date strings into proper Date objects. But when loading from localStorage after a refresh, the code just did `JSON.parse()` and skipped hydration entirely. Dates came back as plain strings and the sort comparison broke.

**What I changed:** In `src/state/store.js`, changed `return JSON.parse(raw)` to `return hydrate(JSON.parse(raw))`. Now hydration always runs no matter where the data is coming from.

---

## Bug 6

**How to reproduce:** Open Balances. Aisha paid $100 for groceries — but the panel says she "owes" money. Carlos, who didn't pay anything, shows as "is owed". Everything is flipped.

**What is wrong:** The labels in `BalancesPanel.jsx` were just backwards. Positive balance means the person paid more than their share, so the group owes them. Negative means they owe the group. The code had it the other way around.

**What I changed:** Swapped the label strings in `src/components/BalancesPanel.jsx`. Positive balance now says "is owed", negative says "owes".

---

## Bug 7

**How to reproduce:** Go to Filters and pick someone from the "Paid by" dropdown, like Aisha. The list goes completely blank even though Aisha definitely paid for stuff.

**What is wrong:** The dropdown gives back the selected value as a string like `"1"`. But `paidBy` on each expense is stored as the number `1`. The filter was comparing `e.paidBy !== paidBy` — a number vs a string — which is never equal in JavaScript. So every single expense got filtered out.

**What I changed:** Changed the comparison in `App.jsx` to `e.paidBy !== Number(paidBy)` so the types actually match before comparing.

---

## Bug 8

**How to reproduce:** Set things up so one person owes exactly what another is owed — say both are $20. The Settle Up panel says "Everyone is settled" even though money is clearly still owed.

**What is wrong:** In `src/lib/settle.js`, the algorithm has three cases: debtor owes more, creditor is owed more, or they match exactly. For the exact match case, it was moving on to the next pair but never recording the transfer. That payment just got silently dropped.

**What I changed:** Added the transfer push inside the equal-amounts case before moving to the next pair. Now when amounts match exactly, the settlement still gets recorded.

---

## Bug 9

**How to reproduce:** Add a new member using the "Add member" form. The "Paid so far" list in Summary still shows only the old members. The new person doesn't appear until you touch an expense.

**What is wrong:** The `perPerson` calculation in `SummaryCards.jsx` is wrapped in a `useMemo`. The dependency array only had `[expenses]` — `members` wasn't listed, so React had no idea it needed to recalculate when a new member was added.

**What I changed:** Added `members` to the dependency array so it's now `[members, expenses]`. Adding a new member now updates the summary immediately like you'd expect.

---

## Bug 10

**How to reproduce:** Switch to "Custom %" split. Enter 33.3, 33.3, 33.4. Hit Save. You get "Percentages must add to 100" even though those numbers add up to 100.

**What is wrong:** Floating point. JavaScript doesn't always land on exactly 100 when adding those up — you can get something like 99.99999999. The code was doing a strict `=== 100` check, so any tiny floating point drift causes it to reject perfectly valid input.

**What I changed:** Changed the check in `src/lib/money.js` to `Math.abs(sum - 100) < 0.01`. Close enough to 100 is good enough.

---

## Bug 11

**How to reproduce:** Open the Add Expense form. The Date field always defaults to 16 March 2026. Doesn't matter what day it actually is.

**What is wrong:** The default date was literally hardcoded as `"2026-03-16"` in `AddExpenseForm.jsx`. Anyone who doesn't notice will log expenses with the wrong date, which also screws up the newest-first sort.

**What I changed:** Replaced `"2026-03-16"` with `new Date().toISOString().slice(0, 10)` so it always defaults to today's actual date.

---

## Bug 12

**How to reproduce:** Edit an expense amount — type a new number and click away to save it. Now click back into that same field. It shows the old amount instead of what you just saved.

**What is wrong:** The edit input has a local `draft` state set with `useState`. `useState` only uses its initial value once when the component mounts — so when the expense updates in the store, `draft` never gets told about it and just keeps showing the stale value.

**What I changed:** Added a `useEffect` in `ExpenseRow` inside `ExpenseList.jsx` that watches `expense.amount` and resets `draft` whenever it changes. Now the input always reflects what's actually saved.

---