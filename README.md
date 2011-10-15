# jasmine-given

jasmine-given is a [Jasmine](https://github.com/pivotal/jasmine) helper that encourages leaner, meaner specs using `Given`, `When`, and `Then`. It is a shameless tribute to Jim Weirich's terrific [rspec-given](https://github.com/jimweirich/rspec-given) gem.

**[Download the latest version here](https://github.com/searls/jasmine-given/archives/master)**.

The basic idea behind the "*-given" meme is a humble acknowledgement of given-when-then as the best English language analogue we have to arrange-act-assert. With rspec and jasmine, we often approximate "given-when-then" with "let-beforeEach-it" (noting that jasmine lacks `let`).

The big idea is "why approximate given-when-then, when we could actually just use them?"

The small idea is "if we couldn't write so English along with our `it` blocks then we'd be encouraged to write cleaner, clearer matchers to articulate our expectations."

Both ideas are pretty cool. Thanks, Jim!

## Example (CoffeeScript)

Oh, and jasmine-given looks *much* nicer in CoffeeScript, so I'll show that example:

``` coffeescript

describe "assigning stuff to this", ->
  Given -> @number = 24
  When -> @number *= 2
  Then -> @number == 48
  # -or-
  Then -> expect(@number).toBe(48)

describe "assigning stuff to variables", ->
  subject=null
  Given -> subject = []
  When -> subject.push('foo')
  Then -> subject.length == 1
  # -or-
  Then -> expect(subject.length).toBe(1)
```

As you might infer from the above, `Then` will trigger a spec failure when the function passed to it returns `false`. As shown above, traditional expectations can still be used, but this can make for significantly easier-to-read expectations when you're asserting something as simple as equality.

## Example (JavaScript)

Of course, jasmine-given also works fine in JavaScript; but as you can see, it's exceptionally clunky in comparison:

``` javascript
describe("assigning stuff to this", function() {
  Given(function() { this.number = 24; });
  When(function() { this.number *= 2; });
  Then(function() { return this.number === 48; });
  Then(function() { expect(this.number).toBe(48) });
});

describe("assigning stuff to variables", function() {
  var subject;
  Given(function() { subject = []; });
  When(function() { subject.push('foo'); });
  Then(function() { return subject.length === 1; });
  Then(function() { expect(subject.length).toBe(1); });
});
```

## Supporting Idempotent "Then" statements

Chatting with Jim earlier this week, he pointed out that `Then` blocks should be idempotent, and not have any affect on the state of the subject being specified. As a result, one optimization that rspec-given could make is to execute **n** `Then` expectations without executing each `Then`'s depended-on `Given` and `When` blocks **n** times.

Take this example from jasmine-given's spec:

``` coffeescript
describe "eliminating redundant test execution", ->
  context "a traditional spec with numerous Then statements", ->
    timesGivenWasInvoked = timesWhenWasInvoked = 0
    Given -> timesGivenWasInvoked++
    When -> timesWhenWasInvoked++
    Then -> timesGivenWasInvoked == 1
    Then -> timesWhenWasInvoked == 2
    Then -> timesGivenWasInvoked == 3
    Then -> timesWhenWasInvoked == 4
```

In the above example, because there are four `Then` statements, the `Given` and `When` are executed four times. That's because it would be unreasonable for Jasmine to expect that each `it` function written in Jasmine is going to be idempotent.

However, we can make that assumption safely when we're writing in a given-when-then format, especially when it's opt-in:

``` coffeescript
  context "chaining Then statements", ->
    timesGivenWasInvoked = timesWhenWasInvoked = 0
    Given -> timesGivenWasInvoked++
    When -> timesWhenWasInvoked++
    Then(-> timesGivenWasInvoked == 1)
    .Then(-> timesWhenWasInvoked == 1)
    .Then(-> timesGivenWasInvoked == 1)
    .Then(-> timesWhenWasInvoked == 1)
```

In this example, `Given` and `When` are only invoked one time each, and the magic that made it possible was to chain `Then` with subsequent `Then` statements. jasmine-given then rolls all of those up into a single `it` in Jasmine.

Leveraging this feature is likely to have the effect of speeding up your specs, especially if you're specs are otherwise slow (integration specs or DOM-heavy).

[Note that in the above, each `Then` needed to be wrapped in parentheses in order for CoffeeScript to understand that we were chaining invocations.]

The above spec is also provided in JavaScript:

``` javascript

describe("eliminating redundant test execution", function() {
  context("a traditional spec with numerous Then statements", function() {
    var timesGivenWasInvoked = 0,
        timesWhenWasInvoked = 0;
    Given(function() { timesGivenWasInvoked++; });
    When(function() { timesWhenWasInvoked++; });
    Then(function() { return timesGivenWasInvoked == 1; });
    Then(function() { return timesWhenWasInvoked == 2; });
    Then(function() { return timesGivenWasInvoked == 3; });
    Then(function() { return timesWhenWasInvoked == 4; });
  });

  context("chaining Then statements", function() {
    var timesGivenWasInvoked = 0,
        timesWhenWasInvoked = 0;
    Given(function() { timesGivenWasInvoked++; });
    When(function() { timesWhenWasInvoked++; });
    Then(function() { return timesGivenWasInvoked == 1; })
    .Then(function() { return timesWhenWasInvoked == 1; })
    .Then(function() { return timesGivenWasInvoked == 1; })
    .Then(function() { return timesWhenWasInvoked == 1; })
  });
});

```