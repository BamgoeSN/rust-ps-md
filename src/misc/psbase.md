# Base Template

## w/o Testcases
```rust,noplayground
use std::{
    io::*,
    str::{FromStr, SplitAsciiWhitespace},
};

fn main() {
    let stdin = stdin();
    let mut handle = stdin.lock();
    let mut input_str = String::new();
    handle
        .read_to_string(&mut input_str)
        .expect("Failed to read");
    let mut scan = Scanner::new(&input_str);
    let stdout = stdout();
    let out = &mut BufWriter::with_capacity(262144, stdout.lock());

    macro_rules! next {
        () => {
            scan.next()
        };
        ($t:ty) => {
            scan.next::<$t>()
        };
    }
    macro_rules! out { ($($arg:tt)*) => { write!(out, $($arg)*).ok(); }; }
    macro_rules! outln { ($($arg:tt)*) => { writeln!(out, $($arg)*).ok(); }; }
}

struct Scanner<'a, I: Iterator<Item = &'a str>> {
    input: I,
}

impl<'a> Scanner<'a, SplitAsciiWhitespace<'a>> {
    fn new(s: &'a str) -> Self {
        Self {
            input: s.split_ascii_whitespace(),
        }
    }
}

impl<'a, I: Iterator<Item = &'a str>> Scanner<'a, I> {
    #[inline(always)]
    fn next<T: ParsableNum>(&mut self) -> T {
        T::from_str(self.next_str())
    }
    #[inline(always)]
    fn next_elem<T: FromStr>(&mut self) -> T {
        self.next_str().parse().ok().expect("Failed to parse")
    }
    #[inline(always)]
    fn next_str(&mut self) -> &'a str {
        self.input.next().expect("Input has been exhausted")
    }
    #[inline(always)]
    fn next_option<T: ParsableNum>(&mut self) -> Option<T> {
        self.next_str_option().map(|s| T::from_str(s))
    }
    #[inline(always)]
    fn next_elem_option<T: FromStr>(&mut self) -> Option<T> {
        self.next_str_option().and_then(|s| s.parse().ok())
    }
    #[inline(always)]
    fn next_str_option(&mut self) -> Option<&'a str> {
        self.input.next()
    }
}

trait ParsableNum {
    fn from_str(s: &str) -> Self;
}
macro_rules! impl_uint { ($($t:ty) *) => {$( impl ParsableNum for $t { fn from_str(s: &str) -> Self { s.bytes().fold(0, |a, b| a * 10 + (b - b'0') as Self) } })+ }; }
impl_uint!(u8 u16 u32 u64 u128 usize);
macro_rules! impl_int { ($($t:ty) *) => { $( impl ParsableNum for $t { fn from_str(s: &str) -> Self { let mut s = s.as_bytes(); let is_neg = s[0] == b'-'; if is_neg {s = &s[1..];} let n = s.iter().fold(0, |a, b| a * 10 + (b - b'0') as Self); if is_neg {-n} else {n} } })+ }; }
impl_int!(i8 i16 i32 i64 i128 isize);
```

## w/ Testcases
```rust,noplayground
use std::{
    io::*,
    str::{FromStr, SplitAsciiWhitespace},
};

fn main() {
    let stdin = stdin();
    let mut handle = stdin.lock();
    let mut input_str = String::new();
    handle
        .read_to_string(&mut input_str)
        .expect("Failed to read");
    let mut scan = Scanner::new(&input_str);
    let stdout = stdout();
    let out = &mut BufWriter::with_capacity(262144, stdout.lock());

    macro_rules! next {
        () => {
            scan.next()
        };
        ($t:ty) => {
            scan.next::<$t>()
        };
    }
    macro_rules! out { ($($arg:tt)*) => { write!(out, $($arg)*).ok(); }; }
    macro_rules! outln { ($($arg:tt)*) => { writeln!(out, $($arg)*).ok(); }; }

    let tc_s: usize = next!();
    for _ in 0..tc_s {}
}

struct Scanner<'a, I: Iterator<Item = &'a str>> {
    input: I,
}

impl<'a> Scanner<'a, SplitAsciiWhitespace<'a>> {
    fn new(s: &'a str) -> Self {
        Self {
            input: s.split_ascii_whitespace(),
        }
    }
}

impl<'a, I: Iterator<Item = &'a str>> Scanner<'a, I> {
    #[inline(always)]
    fn next<T: ParsableNum>(&mut self) -> T {
        T::from_str(self.next_str())
    }
    #[inline(always)]
    fn next_elem<T: FromStr>(&mut self) -> T {
        self.next_str().parse().ok().expect("Failed to parse")
    }
    #[inline(always)]
    fn next_str(&mut self) -> &'a str {
        self.input.next().expect("Input has been exhausted")
    }
    #[inline(always)]
    fn next_option<T: ParsableNum>(&mut self) -> Option<T> {
        self.next_str_option().map(|s| T::from_str(s))
    }
    #[inline(always)]
    fn next_elem_option<T: FromStr>(&mut self) -> Option<T> {
        self.next_str_option().and_then(|s| s.parse().ok())
    }
    #[inline(always)]
    fn next_str_option(&mut self) -> Option<&'a str> {
        self.input.next()
    }
}

trait ParsableNum {
    fn from_str(s: &str) -> Self;
}
macro_rules! impl_uint { ($($t:ty) *) => {$( impl ParsableNum for $t { fn from_str(s: &str) -> Self { s.bytes().fold(0, |a, b| a * 10 + (b - b'0') as Self) } })+ }; }
impl_uint!(u8 u16 u32 u64 u128 usize);
macro_rules! impl_int { ($($t:ty) *) => { $( impl ParsableNum for $t { fn from_str(s: &str) -> Self { let mut s = s.as_bytes(); let is_neg = s[0] == b'-'; if is_neg {s = &s[1..];} let n = s.iter().fold(0, |a, b| a * 10 + (b - b'0') as Self); if is_neg {-n} else {n} } })+ }; }
impl_int!(i8 i16 i32 i64 i128 isize);
```