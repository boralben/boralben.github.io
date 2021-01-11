---
layout: post
title:  "Return Boolean from ViewAssertions"
date:   2021-01-11 12:00:00 -0600
categories: Espresso
---

Espresso has a wonderfully simple syntax for checking the UI behavior of Android apps:

1. Construct a `ViewInteraction` by passing a `ViewMatcher` into `onView`
2. Pass in a `ViewAssertion` to `ViewInteraction`'s `check` function

If the assertion passes, `check` returns the interaction for further use. If the assertion fails, an `AssertionFailedError` is thrown.

For example:

```
onView(withId(R.id.my_label)).check(matches(isDisplayed()))
```

But what if you want to perform a check without breaking the execution of your test. We can use a Kotlin extension on `ViewInteraction` to add a function that will swallow errors and simply return true/false for a given `ViewAssertion`. This sugar allows us to avoid having to wrap our test code in try/catch blocks.

```
fun ViewInteraction.boolCheck(viewAssertion: ViewAssertion): Boolean {
    return try {
        this.check(viewAssertion)
        true
    } catch (e: AssertionFailedError) {
        false
    }
}
```

Usage:

```
onView(withId(R.id.my_label)).boolCheck(matches(isDisplayed()))
```

Another fun Kotlin feature in this code is that the try/catch block is an expression, so we can simply the result of it.

