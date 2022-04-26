# Base Template

## w/o Testcases
```rust,noplayground
use std::{
    io::*,
    str::{FromStr, SplitWhitespace},
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
}

struct Scanner<'a, I: Iterator<Item = &'a str>> {
    input: I,
}

impl<'a> Scanner<'a, SplitWhitespace<'a>> {
    fn new(s: &'a str) -> Self {
        Self {
            input: s.split_whitespace(),
        }
    }
}

impl<'a, I: Iterator<Item = &'a str>> Scanner<'a, I> {
    #[inline(always)]
    fn next<T: FromStr>(&mut self) -> T {
        self.input
            .next()
            .expect("Input has been exhausted")
            .parse()
            .ok()
            .expect("Failed to parse")
    }

    #[inline(always)]
    fn next_option<T: FromStr>(&mut self) -> Option<T> {
        let token = self.input.next();
        match token {
            Some(s) => s.parse().ok(),
            None => None,
        }
    }

    #[inline(always)]
    fn next_str(&mut self) -> &'a str {
        self.input.next().expect("Input has been exhausted")
    }

    #[inline(always)]
    fn next_str_option(&mut self) -> Option<&'a str> {
        self.input.next()
    }
}
```

## w/ Testcases
```rust,noplayground
use std::{
    io::*,
    str::{FromStr, SplitWhitespace},
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

    let tc = next!(usize);
    for _ in 0..tc {}
}

struct Scanner<'a, I: Iterator<Item = &'a str>> {
    input: I,
}

impl<'a> Scanner<'a, SplitWhitespace<'a>> {
    fn new(s: &'a str) -> Self {
        Self {
            input: s.split_whitespace(),
        }
    }
}

impl<'a, I: Iterator<Item = &'a str>> Scanner<'a, I> {
    #[inline(always)]
    fn next<T: FromStr>(&mut self) -> T {
        self.input
            .next()
            .expect("Input has been exhausted")
            .parse()
            .ok()
            .expect("Failed to parse")
    }

    #[inline(always)]
    fn next_option<T: FromStr>(&mut self) -> Option<T> {
        let token = self.input.next();
        match token {
            Some(s) => s.parse().ok(),
            None => None,
        }
    }

    #[inline(always)]
    fn next_str(&mut self) -> &'a str {
        self.input.next().expect("Input has been exhausted")
    }

    #[inline(always)]
    fn next_str_option(&mut self) -> Option<&'a str> {
        self.input.next()
    }
}
```