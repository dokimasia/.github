# dokimasia

One set of test assertions, defined as data, so the same test means the
same thing in every language that implements it.

## The problem

Write the same test twice, once in Python and once in Go, and the two
should pass or fail together. Today they do not.

Go's `cmp` package, configured the way most Go test helpers configure
it, says a nil slice equals an empty slice. Python says `None` does not
equal `[]`. Port a suite from one to the other and both builds go green
while testing two different things. The difference surfaces months later
as a bug that reproduces in one service and not its rewrite.

Six libraries written from a prose specification drift the same way, and
the drift stays invisible because each library's own tests pass. Every
team tests what it believes the specification says. Nothing tests
whether the beliefs agree.

## What is here

The definition is data, not prose, because prose does not fail a build.
Each library vendors a copy and runs the same cases in its own CI.

| Repository | Language | What it is |
|---|---|---|
| [assert-spec](https://github.com/dokimasia/assert-spec) | — | The definition: what each assertion means, what a caller types, and the cases every library runs |
| [assert-go](https://github.com/dokimasia/assert-go) | Go | `go.dokimi.dev/assert` |
| [assert-python](https://github.com/dokimasia/assert-python) | Python | `dokimi-assert` |
| [assert-typescript](https://github.com/dokimasia/assert-typescript) | TypeScript | `@dokimi/assert` |
| [assert-java](https://github.com/dokimasia/assert-java) | Java, Kotlin | `dev.dokimi:assert-core` |
| [assert-rust](https://github.com/dokimasia/assert-rust) | Rust | `dokimi-assert` |

## How a library is held to it

Four mechanisms, because there are four ways to disagree.

**The corpus** states what an assertion means, as data. 87 cases run
against both surfaces in all five libraries. It reaches 25 of the 41
assertions: a case can state its arguments as typed literals, and where
it cannot, it names one of ten behaviours the library builds natively
instead. The remaining 16 want a real machine, a real filesystem or a
real runtime.

**The completeness gate** checks membership. Every assertion, and every
one of the 40 rows naming the rest of the surface, is present under the
name the table gives it, or the library declares in its overlay that it
cannot supply it.

**A failure is a record**, not a sentence. It carries the assertion's
own named values, so a case checks what an assertion reported rather
than searching its prose for a word. Every bug this project has found
across languages hid behind a check that read a sentence.

**Every check is mutated before it ships.** A gate nobody has watched
fail checks nothing. Breaking what a check guards and requiring it to go
red is what caught a seat losing 78% of concurrent failures, a corpus
reader silently ignoring eight files, and two call-site resolvers
pointing at the wrong frame for whole classes of caller.

## What an assertion looks like

Every one takes a seat first and a message last. The seat is where a
failure goes, which is what lets one assertion serve a real test, a
benchmark, and a test that checks the assertion itself.

```go
assert.Equal(t, store.Get(ctx, id), item, "Get returns the stored item")
```

```python
check.equal(seat, store.get(id), item, "get returns the stored item")
```

Two surfaces everywhere: one stops at the first failure, one records and
lets the test carry on, so a single run reports every property that
failed.

## Status

The definition is at 1.0.0 and the five libraries implement it. None is
published to a registry yet, so install from source.

Assertions about behaviour are the part no other library offers:
whether a subject honours cancellation, leaves observed state alone,
survives a missing handle, or stays inside an allocation ceiling.
