# Screw Driver rules explained

Worked examples for each rule. Examples use simple pseudocode that reads like most languages. Translate them into the normal style of the language you are working in.

## Rule 1: Keep logic pure

Pure means same input, same output, nothing else happens. Like a vending machine: same button, same snack.

Bad: the result depends on something hidden.
```
function priceWithTax(price):
    rate = readTaxRateFromDatabase()     # hidden outside read
    return price * (1 + rate)
```

Good: everything it needs comes in as input.
```
function priceWithTax(price, taxRate):
    return price * (1 + taxRate)
```

Why: pure code is easy to test, easy to reuse and never surprises you.

## Rule 2: Pure core, thin shell

Think of a restaurant. The recipe is the core: predictable and written down. The waiter dealing with customers is the shell.

```
# SHELL: talks to the outside world
function handleCheckout(request):
    cart = database.loadCart(request.userId)
    taxRate = config.taxRate()
    result = calculateCheckout(cart, taxRate, today())   # call the core
    if result.isFailure: return errorResponse(result.error)
    database.saveOrder(result.order)
    email.sendReceipt(result.order)
    return successResponse(result.order)

# CORE: plain data in, plain data out
function calculateCheckout(cart, taxRate, date):
    ...
```

Why: all the business logic can be tested without a database, a network or a clock.

## Rule 3: Name every action

Bad: `getUser(id)` that also updates the last login time.
Good: `getUser(id)` only reads. `recordLogin(id)` writes.

Why: a reader should know from the name alone whether calling it is safe.

## Rule 4: Failures are answers, not crashes

Bad: errors thrown from deep inside, or quietly ignored.
```
try:
    chargeCard(card, amount)
catch error:
    # nothing here, failure disappears
```

Good: the result says success or failure, and the caller must decide.
```
function chargeCard(card, amount) -> Success(receipt) or Failure(reason):
    ...

result = chargeCard(card, amount)
if result is Failure:
    log("Charge failed for order " + orderId + ": " + result.reason)
    return showPaymentError(result.reason)
```

Use what the language offers: Result or Either types, error return values, or typed errors. Exceptions only for situations that should be impossible.

## Rule 5: No surprise blanks

Bad: `user.address.city` crashes when the address is missing.
Good: the type says the address may be missing, and the code handles it.
```
city = user.address is present ? user.address.city : "Not provided"
```

Use Optional, Maybe, nullable types with strict checks, or the language's equivalent.

## Rule 6: Make new versions, don't overwrite

Bad:
```
function applyDiscount(order):
    order.total = order.total * 0.9     # changes the caller's data
```

Good:
```
function applyDiscount(order):
    return copy of order with total = order.total * 0.9
```

Why: no more "who changed this and when" bugs.

## Rule 7: Make wrong inputs impossible

Check once at the edge, then use a type that can only hold valid data.
```
# at the edge
emailResult = Email.parse(request.emailText)    # returns Success(Email) or Failure
if emailResult is Failure: return badRequest(emailResult.error)

# in the core, no more checks needed
function sendWelcome(email: Email): ...
```

Also prefer a status type with fixed choices (`Pending`, `Paid`, `Cancelled`) over free text, and avoid pairs of flags that can contradict each other.

## Rule 8: Small blocks that combine

A function that validates, calculates, saves and emails is four functions. Name each one, test each one, then combine them.

## Rule 9: Write the rules before the code

Before writing important logic, write what must always be true:
```
# RULES for calculateCheckout:
# 1. Total is never negative.
# 2. Applying the same discount code twice gives the same total as once.
# 3. Total equals sum of item prices, minus discount, plus tax.
```

These double as acceptance criteria and as the source for rule 10 tests.

## Rule 10: Test the rules, not just examples

An example test checks one case. A property test checks a rule against hundreds of generated inputs and shrinks any failure to the smallest example.

```
property "total is never negative":
    for any random cart and any random discount:
        calculateCheckout(cart, discount).total >= 0
```

Common property testing libraries:
* JavaScript and TypeScript: fast-check
* Python: Hypothesis
* Java and Kotlin: jqwik or Kotest property testing
* Scala: ScalaCheck
* C#: FsCheck
* Go: the built in testing/quick package or rapid
* Rust: proptest
* Swift: SwiftCheck
* Dart and Flutter: glados

If none fits, write a small loop that generates varied inputs.

## Rule 11: Always clean up

Bad: a file is opened, an error happens, the file is never closed.
Good: use the language's safe pattern so closing always happens.
```
with open(file) as f:        # closes even if an error happens
    process(f)
```

Equivalents: try with finally, using, defer, try with resources, or automatic cleanup when an object goes out of scope.
