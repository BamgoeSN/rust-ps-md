# Base Template

## w/o Testcases
```rust,noplayground
use std::{
    io::*,
    str::{FromStr, SplitWhitespace},
};

fn main() {
    let stdin = std::fs::File::open("input.txt").ok().unwrap();
    let mut handle = BufReader::new(stdin);
    // let stdin = stdin();
    // let mut handle = stdin.lock();
    let mut input_str = String::new();
    handle
        .read_to_string(&mut input_str)
        .expect("Failed to read");
    let mut scan = Scanner::new(&input_str);
    let out = &mut BufWriter::new(stdout());
}

struct Scanner<'a> {
    input: SplitWhitespace<'a>,
}

impl<'a> Scanner<'a> {
    fn new(s: &'a str) -> Self {
        Self {
            input: s.split_whitespace(),
        }
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next<T: FromStr>(&mut self) -> T {
        self.input
            .next()
            .expect("Input has been exhausted")
            .parse()
            .ok()
            .expect("Failed to parse")
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next_option<T: FromStr>(&mut self) -> Option<T> {
        let token = self.input.next();
        match token {
            Some(s) => s.parse().ok(),
            None => None,
        }
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next_str(&mut self) -> &'a str {
        self.input.next().expect("Input has been exhausted")
    }

    #[allow(dead_code)]
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
    let stdin = std::fs::File::open("input.txt").ok().unwrap();
    let mut handle = BufReader::new(stdin);
    // let stdin = stdin();
    // let mut handle = stdin.lock();
    let mut input_str = String::new();
    handle
        .read_to_string(&mut input_str)
        .expect("Failed to read");
    let mut scan = Scanner::new(&input_str);
    let out = &mut BufWriter::new(stdout());

    let tc: usize = scan.next();
    for _ in 0..tc {}
}

struct Scanner<'a> {
    input: SplitWhitespace<'a>,
}

impl<'a> Scanner<'a> {
    fn new(s: &'a str) -> Self {
        Self {
            input: s.split_whitespace(),
        }
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next<T: FromStr>(&mut self) -> T {
        self.input
            .next()
            .expect("Input has been exhausted")
            .parse()
            .ok()
            .expect("Failed to parse")
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next_option<T: FromStr>(&mut self) -> Option<T> {
        let token = self.input.next();
        match token {
            Some(s) => s.parse().ok(),
            None => None,
        }
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next_str(&mut self) -> &'a str {
        self.input.next().expect("Input has been exhausted")
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next_str_option(&mut self) -> Option<&'a str> {
        self.input.next()
    }
}
```