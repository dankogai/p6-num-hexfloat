[![.github/workflows/test.yml](https://github.com/dankogai/p6-num-hexfloat/actions/workflows/test.yml/badge.svg)](https://github.com/dankogai/p6-num-hexfloat/actions/workflows/test.yml)

# NAME

Num::HexFloat - Rudimentary C99 Hexadecimal Floating Point Support in Perl6

## STATUS: MOSTLY OBSOLETE

Rakudo has grown native C99 hexadecimal floating point support.  It is
**not** in the 2026.07 release, but it is in `main` as of 2026-08-07 and
is expected to ship in 2026.08:

* [`0860b287c`](https://github.com/rakudo/rakudo/commit/0860b287c) —
  hexadecimal float *literals* (`0x1.8p+1`), in both the legacy and the
  RakuAST grammars.
* [`999492263`](https://github.com/rakudo/rakudo/commit/999492263) /
  [Raku/nqp#859](https://github.com/Raku/nqp/pull/859) — `%a` / `%A`
  directives in `sprintf`, `printf` and `.fmt`.
* [`669ad276d`](https://github.com/rakudo/rakudo/commit/669ad276d) —
  the same `%a` / `%A` directives in the RakuAST `Formatter` that
  becomes the `sprintf` engine under `6.e`.

So on a recent enough Rakudo you no longer need this module for the
common cases:

````perl6
say 0x1.921fb54442d18p+1 === pi;    # True   -- was from-hexfloat(...)
say pi.fmt("%a");                   # 0x1.921fb54442d18p+1
                                    #        -- was to-hexfloat(pi)
say sprintf("%.3a", pi);            # 0x1.922p+1
````

### What is still not native

Rakudo parses hexadecimal floats **at compile time only**.  There is no
runtime `Str` → `Num` conversion — `p+1` still looks like trailing
garbage to the numeric grammar, so `"0x1.8p+1".Num` and `+"0x1.8p+1"`
throw `X::Str::Numeric` ("trailing characters after number"), and
`val("0x1.8p+1")` declines to numify and hands back a plain `Str`.

That leaves exactly one job for this module — digging hexadecimal floats
out of text you did not write yourself:

````perl6
my $src = "e=0x1.5bf0a8b145769p+1, pi=0x1.921fb54442d18p+1";
say $src.subst($RE_HEXFLOAT, &from-hexfloat, :g);
````

If your hexfloats are literals in your source, or you only need to
*print* them, drop the dependency and use the built-ins.

### Differences from the built-ins

`to-hexfloat` and `%a` agree byte-for-byte on every finite value,
including subnormals and the extremes (`0x1p-1074`,
`0x1.fffffffffffffp+1023`).  They differ only on the non-finite ones:
this module follows C and prints `inf` / `-inf` / `nan`, while `%a`
follows Raku and prints `Inf` / `-Inf` / `NaN`, as `%e`, `%f` and `%g`
already do.

The built-in `%a` also supports the C flags this module never had:
precision (`%.3a`), uppercase (`%A`), width and zero padding (`%016a`,
which pads between the `0x` and the mantissa), and `#` to keep the radix
point.

## SYNOPSIS

````perl6
use v6;
use Num::HexFloat;
   
say to-hexfloat(pi);
# '0x1.921fb54442d18p+1'
say from-hexfloat('0x1.921fb54442d18p+1') == pi;
# True
my $src = "e=0x1.5bf0a8b145769p+1, pi=0x1.921fb54442d18p+1";
say $src.subst($RE_HEXFLOAT, &from-hexfloat, :g);
# e=2.71828182845905, pi=3.14159265358979
````

## DESCRIPTION

`Num::HexFloat` exports the following:

### `$RE_HEXFLOAT`

A regex that matches hexadecimal floating point notation.

### `from-hexfloat($arg) returns Num`

Parses `$arg` as a C99 hexadecimal floating point notation and returns
`Num`, or `NaN` if it fails.

`$arg` can be either `Str` or `Match` so you can go like: 

````perl6
$src.subst($RE_HEXFLOAT, &from-hexfloat, :g);
````

### `to-hexfloat(Numeric $num) returns Str`

Stringifies `$num` in C99 hexadecimal floating point notation.

