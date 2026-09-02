# Bugs found

Add one section per issue. Bug 1 is filled in to show the format - fix it, then write what you changed. Copy the blank template for the rest.

Keep this file in the repo and **commit it** with your fixes.

---

## Bug 1

**How to reproduce:** Open the app. The expense list says "Newest first". The first row is Wine (7 Mar). Board game (15 Mar) is further down.

**What is wrong:** The list is showing oldest expenses first. Newest should be at the top.

**What I changed:** In `src/components/ExpenseList.jsx` line 62, the sort comparator was `dateValue(a.date) - dateValue(b.date)` (ascending — oldest first). Swapped it to `dateValue(b.date) - dateValue(a.date)` (descending — newest first) so Board game (15 Mar) now appears at the top and Wine (7 Mar) at the bottom.

---

## Bug 2

**How to reproduce:**

**What is wrong:**

**What I changed:**

---