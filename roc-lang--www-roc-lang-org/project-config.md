---
trigger: always_on
description: See README.md for build instructions.
---

## project specific instructions

See README.md for build instructions.

## general roc instructions

Note that the instructions below are for Roc alpha 4 not the new zig compiler version of Roc, Roc alpha 4 has limited use in this repo, like in build_website.roc.

After you've made changes use `roc check edited_file.roc` to confirm it is free of errors.
Once `roc check` succeeds, you can run code with `roc file.roc`.
To run all top level expects, use `roc test file.roc`.

The Roc stdlib supports these functions (require no import):
```
Str.Utf8Problem : [ InvalidStartByte, UnexpectedEndOfSequence, ExpectedContinuation, OverlongEncoding, CodepointTooLarge, EncodesSurrogateHalf ]
Str.is_empty : Str -> Bool
Str.concat : Str, Str -> Str
Str.with_capacity : U64 -> Str
Str.reserve : Str, U64 -> Str
Str.join_with : List Str, Str -> Str
Str.split_on : Str, Str -> List Str
Str.repeat : Str, U64 -> Str
Str.len : Str -> [LearnAboutStringsInRoc Str]
Str.to_utf8 : Str -> List U8
Str.from_utf8 : List U8 -> Result Str [ BadUtf8 { problem : Utf8Problem, index : U64 } ]
Str.from_utf8_lossy : List U8 -> Str
Str.from_utf16 : List U16 -> Result Str [ BadUtf16 { problem : Utf8Problem, index : U64 } ]
Str.from_utf16_lossy : List U16 -> Str
Str.from_utf32 : List U32 -> Result Str [ BadUtf32 { problem : Utf8Problem, index : U64 } ]
Str.from_utf32_lossy : List U32 -> Str
Str.starts_with : Str, Str -> Bool
Str.ends_with : Str, Str -> Bool
Str.trim : Str -> Str
Str.trim_start : Str -> Str
Str.trim_end : Str -> Str
Str.to_dec : Str -> Result Dec [InvalidNumStr]
Str.to_f64 : Str -> Result F64 [InvalidNumStr]
Str.to_f32 : Str -> Result F32 [InvalidNumStr]
Str.to_u128 : Str -> Result U128 [InvalidNumStr]
Str.to_i128 : Str -> Result I128 [InvalidNumStr]
Str.to_u64 : Str -> Result U64 [InvalidNumStr]
Str.to_i64 : Str -> Result I64 [InvalidNumStr]
Str.to_u32 : Str -> Result U32 [InvalidNumStr]
Str.to_i32 : Str -> Result I32 [InvalidNumStr]
Str.to_u16 : Str -> Result U16 [InvalidNumStr]
Str.to_i16 : Str -> Result I16 [InvalidNumStr]
Str.to_u8 : Str -> Result U8 [InvalidNumStr]
Str.to_i8 : Str -> Result I8 [InvalidNumStr]
Str.count_utf8_bytes : Str -> U64
Str.replace_each : Str, Str, Str -> Str
Str.replace_first : Str, Str, Str -> Str
Str.replace_last : Str, Str, Str -> Str
Str.split_first : Str, Str -> Result { before : Str, after : Str } [NotFound]
Str.split_last : Str, Str -> Result { before : Str, after : Str } [NotFound]
Str.walk_utf8_with_index : Str, state, (state, U8, U64 -> state) -> state
Str.walk_utf8 : Str, state, (state, U8 -> state) -> state
Str.release_excess_capacity : Str -> Str
Str.with_prefix : Str, Str -> Str
Str.contains : Str, Str -> Bool
Str.drop_prefix : Str, Str -> Str
Str.drop_suffix : Str, Str -> Str
Str.with_ascii_lowercased : Str -> Str
Str.with_ascii_uppercased : Str -> Str
Str.caseless_ascii_equals : Str, Str -> Bool
Num.e : Frac *
Num.pi : Frac *
Num.tau : Frac *
Num.to_str : Num * -> Str
Num.int_cast : Int a -> Int b
Num.compare : Num a, Num a -> [ LT, EQ, GT ]
Num.is_lt : Num a, Num a -> Bool
Num.is_gt : Num a, Num a -> Bool
Num.is_lte : Num a, Num a -> Bool
Num.is_gte : Num a, Num a -> Bool
Num.is_approx_eq : Frac a, Frac a, { rtol ? Frac a, atol ? Frac a } -> Bool
Num.is_zero : Num a -> Bool
Num.is_even : Int a -> Bool
Num.is_odd : Int a -> Bool
Num.is_positive : Num a -> Bool
Num.is_negative : Num a -> Bool
Num.to_frac : Num * -> Frac *
Num.is_nan : Frac * -> Bool
Num.is_infinite : Frac * -> Bool
Num.is_finite : Frac * -> Bool
Num.abs : Num a -> Num a
Num.abs_diff : Num a, Num a -> Num a
Num.neg : Num a -> Num a
Num.add : Num a, Num a -> Num a
Num.sub : Num a, Num a -> Num a
Num.mul : Num a, Num a -> Num a
Num.min : Num a, Num a -> Num a
Num.max : Num a, Num a -> Num a
Num.sin : Frac a -> Frac a
Num.cos : Frac a -> Frac a
Num.tan : Frac a -> Frac a
Num.asin : Frac a -> Frac a
Num.acos : Frac a -> Frac a
Num.atan : Frac a -> Frac a
Num.sqrt : Frac a -> Frac a
Num.sqrt_checked : Frac a -> Result (Frac a) [SqrtOfNegative]
Num.log : Frac a -> Frac a
Num.log_checked : Frac a -> Result (Frac a) [LogNeedsPositive]
Num.div : Frac a, Frac a -> Frac a
Num.div_checked : Frac a, Frac a -> Result (Frac a) [DivByZero]
Num.div_ceil : Int a, Int a -> Int a
Num.div_ceil_checked : Int a, Int a -> Result (Int a) [DivByZero]
Num.div_trunc : Int a, Int a -> Int a
Num.div_trunc_checked : Int a, Int a -> Result (Int a) [DivByZero]
Num.rem : Int a, Int a -> Int a
Num.rem_checked : Int a, Int a -> Result (Int a) [DivByZero]
Num.is_multiple_of : Int a, Int a -> Bool
Num.bitwise_and : Int a, Int a -> Int a
Num.bitwise_xor : Int a, Int a -> Int a
Num.bitwise_or : Int a, Int a -> Int a
Num.bitwise_not : Int a -> Int a
Num.shift_left_by : Int a, U8 -> Int a
Num.shift_right_by : Int a, U8 -> Int a
Num.shift_right_zf_by : Int a, U8 -> Int a
Num.round : Frac * -> Int *
Num.floor : Frac * -> Int *
Num.ceiling : Frac * -> Int *
Num.pow : Frac a, Frac a -> Frac a
Num.pow_int : Int a, Int a -> Int a
Num.count_leading_zero_bits : Int a -> U8
Num.count_trailing_zero_bits : Int a -> U8
Num.count_one_bits : Int a -> U8
Num.add_wrap : Int range, Int range -> Int range
Num.add_saturated : Num a, Num a -> Num a
Num.add_checked : Num a, Num a -> Result (Num a) [Overflow]
Num.sub_wrap : Int range, Int range -> Int range
Num.sub_saturated : Num a, Num a -> Num a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roc-lang/www.roc-lang.org](https://github.com/roc-lang/www.roc-lang.org) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
