---
name: screwdriver
description: Coding standard for writing simple, easy to fix, failure safe code in any language. Use whenever writing, changing, refactoring or reviewing code, starting a new project, or when asked for a code review. Principles inspired by the book Functional Programming in Scala, applied to every language.
---

# Screw Driver

Write code that is simple, easy to fix, and never fails silently. These rules work in any programming language. Apply them using the normal style and tools of the language you are working in.

## Step 1: Pick the mode (once per project)

Look in the project's CLAUDE.md for a line that says `Screw Driver mode: guided` or `Screw Driver mode: expert`.

If there is no such line, ask the user once:
"Do you want me to follow your existing code style (expert mode), or apply the full Screw Driver standard (guided mode)?"
Then save the answer as that line in CLAUDE.md, creating the file if needed. Never ask again in this project.

If you cannot ask (no user present): use guided mode for a new or empty project, expert mode for an existing codebase with a clear style. Say which one you picked in your report.

The user can switch at any time by saying "switch to guided mode" or "switch to expert mode". Update CLAUDE.md when they do.

**Guided mode:** Apply every rule below fully. Explain your work in plain English for a reader who does not code.

**Expert mode:** The project's existing style and patterns come first. Do not rewrite working code to fit these rules. Suggest improvements in your report instead. Still always flag rule 4 (silent failures), rule 5 (surprise blanks) and rule 11 (missing cleanup), because those cause real outages.

## Step 2: Follow the 11 rules while writing

1. **Keep logic pure.** Calculation and decision code returns the same output for the same input and does nothing else. No hidden reads or writes.
2. **Pure core, thin shell.** Anything that touches the outside world (database, network, files, user input, logging, current time, random numbers, environment settings) lives in a thin outer layer. The core receives plain data and returns plain data.
3. **Name every action.** A function that charges, saves, sends, deletes or changes anything outside itself says so in its name (`saveOrder`, `sendReceiptEmail`). A function named like a question or a calculation never changes anything.
4. **Failures are answers, not crashes.** Expected failures (bad input, not found, network down, payment declined) are returned as clear results the caller must handle, using the language's normal way to do that. Never swallow an error. Never use an empty catch block. Reserve crashing exceptions for truly impossible situations, and give every error message enough detail to fix it.
5. **No surprise blanks.** If a value can be missing, the code says so in its type or signature and every caller handles the missing case. No unchecked null or undefined.
6. **Make new versions, don't overwrite.** Treat data as read only. Return an updated copy instead of changing the original. Exception: changing data inside one function is fine when nothing outside can see it and speed truly matters. Comment why.
7. **Make wrong inputs impossible.** Check and convert outside data once, at the edge, into precise types. Inside the core, trust the types. Prefer types that cannot hold an invalid state over checks scattered everywhere.
8. **Small blocks that combine.** Each function does one job and fits on one screen. Build bigger behaviour by combining small, tested pieces.
9. **Write the rules before the code.** For any important logic (money, permissions, data changes, business rules), first write in a comment what must always be true, for example "total is never negative" or "cancelling twice equals cancelling once".
10. **Test the rules, not just examples.** Turn the rules from step 9 into property tests that run many generated inputs, plus a few normal example tests. In guided mode, add the standard property testing library for the language automatically and mention it in the report. In expert mode, ask before adding a new library.
11. **Always clean up.** Every file, connection, lock or subscription that is opened is closed on every path, including failures. Use the language's built in safe pattern for this.

Always on, in both modes:
* Clear comments that explain why, not what, for future readers.
* A README section called "How this project works", written for a non coder, kept up to date as the project grows.
* Prefer the simplest code that meets the rules. No clever tricks, no unneeded abstractions, no libraries that are not needed.

For a worked example of any rule, open `references/rules-explained.md` if it exists. Only open it when you are unsure how to apply a rule. Do not load it by default.

## Step 3: Self check before finishing

Check only the code you added or changed in this task. Do not reread the whole project.

1. Is every calculation free of hidden outside effects? (rules 1, 2)
2. Does every function that changes something say so in its name? (rule 3)
3. Is every expected failure returned and handled, with no empty catch? (rule 4)
4. Is every possibly missing value marked and handled? (rule 5)
5. Is shared data left unchanged, with copies returned instead? (rule 6)
6. Is outside input checked once at the edge? (rule 7)
7. Does each function do one job? (rule 8)
8. Are important rules written down and tested? (rules 9, 10)
9. Is everything opened also closed, even on failure? (rule 11)
10. Are there clear comments, and is the README section still accurate?

Fix anything that fails before reporting. Mention only what you could not fix.

## Step 4: Report

Keep your phase reports to less than 4 lines unless something needs my decision.

Use this shape, in plain English:
**Done:** what changed.
**Why:** the reason, in one line.
**Could fail:** what might go wrong and how the code handles it.
**Needs you:** a decision, only if there is one.

## When to relax

* The user's direct instruction always wins over these rules.
* For a throwaway script or a quick experiment the user labels as such, apply rules 4, 5 and 11 only.

## Full review on request

When the user asks to review the whole project, apply the Step 3 checklist to every file, then report findings grouped by rule, most serious first, each with a one line fix.
