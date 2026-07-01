# Search Syntax

The asset search box supports a small query language: plain text, field
filters, comparison operators, and boolean logic with grouping. Matching is
always case-insensitive.

## Basic text search

| Query | Meaning |
|-------|---------|
| `zombie` | Name **contains** `zombie` (case-insensitive) |
| `zombie soldier` | Name contains `zombie` **AND** contains `soldier` (space = AND) |
| `"cold war"` | Quoted phrase — spaces/operators inside are literal. Single quotes `'...'` also work; `\` escapes the next char |

## Text match modes

Applied to plain terms and to `name:` / `bonename:` values:

| Prefix/suffix | Mode | Example | Matches |
|---------------|------|---------|---------|
| *(none)* | Contains | `head` | `...head...` |
| `+value` | Exact | `+head` | exactly `head` |
| `value...` | Starts with | `head...` | `head*` |
| `...value` | Ends with | `...head` | `*head` |
| `...value...` | Exact | `...head...` | exactly `head` |
| `-value` | Invert (negate) this term | `-head` | name does **not** contain `head` |

`-` and `+` can be combined/stacked; `-` toggles negation each time.

## Fields

`field:value` (or `field=value`). Field names are case-insensitive:

| Field | Aliases | Value type |
|-------|---------|-----------|
| Name | `name`, `n` | text |
| Bone name | `bonename`, `bone` | text (matches if **any** bone matches) |
| Hash | `hash`, `h` | number (decimal or `0x` hex) |
| Bone count | `bonecount` | number |
| Mesh count | `meshcount` | number |
| Frame count | `framecount` | number |
| Frame rate | `framerate`, `fps` | number |
| Width | `width` | number |
| Height | `height` | number |
| Channels | `channels` | number |
| Kind | `kind`, `type`, `k` | kind alias(es) |

If a `field:value` doesn't resolve to a known field (or the value doesn't
parse), it falls back to a plain name-contains search on the literal text.

## Comparison operators

Numeric fields (`width`, `bonecount`, `hash`, etc.) accept:

| Operator | Meaning |
|----------|---------|
| `:` or `=` or `==` | equal |
| `!=` or `!:` | not equal |
| `<` | less than |
| `<=` or `<:` | less than or equal |
| `>` | greater than |
| `>=` or `>:` | greater than or equal |

Examples: `width:1024`, `bonecount>=50`, `framerate<30`, `meshcount!=0`,
`hash:0x1a2b3c`.

Text fields (`name`, `bonename`) only support `:` / `=` (contains) and `!=`
(not contains) — use the prefix/suffix modes above for starts/ends/exact.

## Kind filter

`kind:` maps aliases to a stable asset-type tag:

| Aliases | Kind |
|---------|------|
| `animation`, `anim`, `a` | Animation |
| `image`, `img`, `i`, `texture`, `tex` | Image |
| `model`, `mdl`, `m`, `mesh` | Model |
| `sound`, `snd`, `audio`, `s` | Sound |
| `material`, `mat` | Material |
| `rawfile`, `raw`, `file`, `r` | Raw file |
| `movie`, `mov`, `video` | Movie |
| `map` | Map |
| `camo` | Camo |
| `weapon`, `wep` | Weapon |
| `sndbank`, `bank` | Sound bank |
| `animpkg`, `pkg` | Anim package |

- Comma-separated list = match any: `kind:model,anim`
- Negate: `kind!=sound`

## Boolean logic & grouping

| Operator | Forms |
|----------|-------|
| AND | `AND`, `&&`, `&`, or just a space between terms |
| OR | `OR`, `||`, `|`, `,` |
| NOT | `NOT`, `!` (prefix) |
| Group | `( ... )` |

AND binds tighter than OR. Examples:

```
kind:model AND bonecount>50
head, hands, feet                    # OR: any of the three
weapon (ak47 OR m4)                  # kind weapon and one of two names
!kind:sound framerate>=30            # NOT sound, and fps >= 30
name:zombie AND NOT bonename:tag_    # zombie models without a tag_ bone
(width:1024 OR width:2048) kind:image
```

## Hash search

Plain name terms also match against the asset's **hash** when a hash seed is
active: a term matches if the name contains it **or** the FNV1a-64 hash of the
term (masked to 63 bits) equals the asset/bone hash. You can also query the
hash directly: `hash:0xABCDEF12` or `hash:123456789`.

## Notes

- Everything is case-insensitive.
- Unmatched/incomplete input degrades gracefully rather than erroring — e.g. a
  dangling operator, empty group, or unknown field falls back to plain text
  matching or is ignored.
