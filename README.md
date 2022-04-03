# Duplicate named capturing groups

Right now, named capturing groups in a regex need to be unique:

```js
str.match(/(?<year>[0-9]{4})-[0-9]{2}|[0-9]{2}-(?<year>[0-9]{4})/)
```

is an error, because you've reused the `year` group name. But sometimes you want to match a thing which can be written in multiple formats (as above). It would be nice to be able to use the same group name for that case.

## Spec text

See [this PR](https://github.com/tc39/ecma262/pull/2721).

## Other languages

Per [this webpage](https://www.regular-expressions.info/named.html#duplicate), several languages allow duplicate names, although with slightly varying semantics.

TODO: better document exactly how it works in various languages: in what case is it legal, and how does clobbering work if that's relevant?

## Open question: only allowed across alternatives?

Should this only allow you to re-use names when they occur in different `|` alternatives, such that it's impossible for a single match to actually use the same name multiple times? Or should it be allowed more generally with some sort of clobbering behavior?

My preference is to only allow it when in different alternatives. That's the only use case I'm aware of, and the error is still helpful if you mess that up.

### Last wins?

If we _do_ allow names to be re-used within an alternative, should it be first-wins or last-wins?

Other langues do first-wins, apparently. On the other hand, if you have something like `/(.)+/.exec("ab")` (where a group is repeated with `+` or `*`), it's last-wins: in that example the `1` property holds `b`, not `a`.

## Previous discussion

See [this issue](https://github.com/tc39/proposal-regexp-named-groups/issues/44) and [some discussion on twitter](https://twitter.com/littledan/status/1352019266300768266).

## Proposal or needs-consensus PR?

It feels like there is not much design space here, so possibly this could be a needs-consensus PR.
