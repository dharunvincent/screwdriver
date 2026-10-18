# Screw Driver 🪛

**Coding principles from *Functional Programming in Scala*, for Claude Code. Any language.**

Screw Driver is a skill for Claude Code that makes Claude write code that is simple, easy to fix, and never fails silently. It is built for two kinds of people:

* **Product managers and builders who don't code** and want Claude to produce solid, well explained projects.
* **Engineers** who want a consistent standard, without Claude forcing a style onto a codebase that already works.

## Why this exists

Claude already knows good coding principles. It just doesn't apply them every time. Screw Driver turns a handful of time tested ideas into rules Claude follows on every task, plus a quick self check before it says "done".

## Before and after

A checkout function, the way code often gets written:

```javascript
// BEFORE: everything mixed together
async function checkout(userId) {
  const cart = await db.getCart(userId);
  let total = 0;
  for (const item of cart.items) total += item.price;
  if (cart.coupon) total = total - 10;        // can go negative
  try {
    await payments.charge(userId, total);
  } catch (e) {}                              // failure silently ignored
  await email.send(userId, "Thanks!");        // sent even if payment failed
  return total;
}
```

The same thing with Screw Driver:

```javascript
// AFTER: pure core, thin shell, failures handled

// CORE: plain data in, plain data out. Easy to test.
// RULE: total is never negative.
function calculateTotal(items, coupon) {
  const subtotal = items.reduce((sum, item) => sum + item.price, 0);
  const discount = coupon ? coupon.amount : 0;
  return Math.max(0, subtotal - discount);
}

// SHELL: the only part that talks to the outside world.
async function checkout(userId) {
  const cart = await db.getCart(userId);
  if (!cart) return { ok: false, error: "Cart not found" };

  const total = calculateTotal(cart.items, cart.coupon);

  const payment = await chargeCard(userId, total);
  if (!payment.ok) return { ok: false, error: `Payment failed: ${payment.reason}` };

  await sendReceiptEmail(userId, total);       // only after payment succeeds
  return { ok: true, total };
}
```

The total can't go negative, payment failures are never hidden, and the receipt only goes out after a successful charge.

## The 11 rules

| # | Rule | In plain English |
|---|------|------------------|
| 1 | Keep logic pure | Same input, same output, nothing hidden |
| 2 | Pure core, thin shell | Business logic in the middle, outside world at the edges |
| 3 | Name every action | If it saves, sends or deletes, the name says so |
| 4 | Failures are answers | Every failure is returned and handled, never swallowed |
| 5 | No surprise blanks | If something can be missing, the code says so upfront |
| 6 | New versions, not overwrites | Data is copied and updated, not changed in place |
| 7 | Wrong inputs impossible | Check outside data once, at the door |
| 8 | Small blocks | Each function does one job |
| 9 | Rules before code | Write down what must always be true |
| 10 | Test the rules | Property tests with many generated inputs |
| 11 | Always clean up | Everything opened gets closed, even on failure |

Worked examples for every rule live in [references/rules-explained.md](references/rules-explained.md).

## Two modes

* **Guided mode:** every rule applied fully, with plain English explanations. Best for new projects and non coders.
* **Expert mode:** your existing style comes first. Claude only suggests changes, but still flags silent failures, unhandled missing values and missing cleanup.

Claude asks once per project and saves your choice in `CLAUDE.md`. You can switch anytime by saying "switch to expert mode" or "switch to guided mode".

## What you get after every task

A short report of less than 4 lines, unless something needs your decision:

```
Done: Added checkout with discount support.
Why: Users asked for coupon codes.
Could fail: Payment declined; the user sees a clear message and no receipt is sent.
```

## Install

Claude Code loads personal skills from `~/.claude/skills/`.

**Option 1: clone straight into your skills folder**

```bash
git clone https://github.com/YOUR_USERNAME/screwdriver.git ~/.claude/skills/screwdriver
```

**Option 2: keep the repo elsewhere and link it** (best if you want to edit it)

```bash
git clone https://github.com/YOUR_USERNAME/screwdriver.git ~/Projects/screwdriver
ln -s ~/Projects/screwdriver ~/.claude/skills/screwdriver
```

Claude Code picks up new skills automatically. If it doesn't, run `/reload-skills` or restart Claude Code. To check it's installed, ask Claude: "What skills do you have?"

To update later: `git pull` inside the folder.

## Credits

The ideas behind Screw Driver come from **[Functional Programming in Scala](https://www.manning.com/books/functional-programming-in-scala)** by **Paul Chiusano and Rúnar Bjarnason** (Manning). Everything here is explained in my own words and applied to every language. If you want the deep version, read the book.

This project is not affiliated with or endorsed by the authors, Manning, the Scala project, or Anthropic.

## Contributing

Contributions are welcome, especially:

* Language notes showing how each rule looks in your favourite language
* Better examples in `references/rules-explained.md`
* Real cases where a rule caused trouble, so it can be improved

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT. Free to use, change and share. See [LICENSE](LICENSE).

Made by [Dharun Vincent](https://dharunvincent.com).
