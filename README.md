# Duplicate named capturing groups

Right now, in JavaScript, named capturing groups in a regex need to be unique. So

```js
str.match(/(?<year>[0-9]{4})-[0-9]{2}|[0-9]{2}-(?<year>[0-9]{4})/)
```

is an error, because you've reused the `year` group name. But sometimes you want to match a thing which can be written in multiple formats (as above). It would be nice to be able to use the same group name for that case.

## Status

This proposal achieved stage 3 at [the July 2022 meeting](https://github.com/tc39/agendas/blob/main/2022/07.md).

Implementations are in progress. See [this issue](https://github.com/tc39/proposal-duplicate-named-capturing-groups/issues/4) to track current status. As of this writing only Safari has implemented.

## Spec text

See [this PR](https://github.com/tc39/ecma262/pull/2721). Spec text should be complete.

## Tests

Tests262 tests are available in [this PR](https://github.com/tc39/test262/pull/3625).

## Details

### Duplicate names must be in different alternatives

At the moment this proposal only allow you to re-use names when they occur in different `|` alternatives, so that it's impossible for a single match to actually use the same name multiple times. If you have a use case for re-using a name within a single alternative, open an issue. Note that this would make the semantics much more complicated.

### Backreferences

With this change it becomes possible to have a backreference which might refer to multiple different groups, as in

```js
/(?:(?<a>x)|(?<a>y))\k<a>/
```

However, the restriction above means that at most one of the groups can actually have participated in the match.\*

So at most one of those actually will have matched. This proposal says that the backreference refers to whichever actually participated. In the case that none have matched, it unconditionally matches without consuming any input (which is already how it works).

\*Technically it's possible for different groups to have participated on different iterations of a repetition (`+`, `*`, `{2}`, etc), but it's already established that only the last iteration "counts" for the purposes of backreferences.

### The `groups` object

As with backreferences, the corresponding named property of the `groups` object on a match result refers to whichever of the groups actually participated in the match.

For the purposes of property enumeration order, these named properties are created in the order in which they appear in the regex, regardless of which groups actually participated in the match.

### Can you _refer_ to a group in a different alternative?

Consider

```js
/(?<a>x)|\k<a>/
```

The backreference is never useful, because the group to which it refers is in a different alternative. So its behavior is to always match without consuming any input. (That's weird but also how it works for `\1`-style matches.) This is true even with repetitions, because capturing groups within a repeated section are cleared at the start of each iteration. As such there's no reason to ever write this.

Unfortunately, it's currently legal. This proposal would not change that.

## Tests

None yet. PRs welcome!

## Other languages

Per [this webpage](https://www.regular-expressions.info/named.html#duplicate), several languages allow duplicate names, although with slightly varying semantics. I believe that restricting this to groups in different alternatives keeps this proposal in the intersection of their semantics.

## Previous discussion

See [this issue](https://github.com/tc39/proposal-regexp-named-groups/issues/44) and [some discussion on twitter](https://twitter.com/littledan/status/1352019266300768266).

