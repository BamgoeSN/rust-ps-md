# Base Template

## w/o Testcases
```rust,noplayground
use std::io::*;

fn main() {
    let stdin = std::fs::File::open("input.txt").ok().unwrap();
    let handle = BufReader::new(stdin);
    // let stdin = stdin();
    // let handle = stdin.lock();
    let mut scan = Scanner::new(handle);
    let out = &mut BufWriter::new(stdout());
}

struct Scanner<R> {
    _reader: R,
    buffer: Vec<String>,
}

impl<R> Scanner<R>
where
    R: BufRead,
{
    fn new(mut reader: R) -> Self {
        let mut input = String::new();
        reader.read_to_string(&mut input).expect("Failed to read");

        let mut buffer = Vec::new();
        buffer.extend(input.split_whitespace().rev().map(String::from));

        Self {
            _reader: reader,
            buffer,
        }
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next<T>(&mut self) -> T
    where
        T: std::str::FromStr,
    {
        match self.buffer.pop() {
            Some(token) => token.parse().ok().expect("Failed to parse"),
            None => panic!("Input is all drained"),
        }
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next_option<T>(&mut self) -> Option<T>
    where
        T: std::str::FromStr,
    {
        match self.buffer.pop() {
            Some(token) => token.parse().ok(),
            None => None,
        }
    }
}
```

## w/ Testcases
```rust,noplayground
use std::io::*;

fn main() {
    let stdin = std::fs::File::open("input.txt").ok().unwrap();
    let handle = BufReader::new(stdin);
    // let stdin = stdin();
    // let handle = stdin.lock();
    let mut scan = Scanner::new(handle);
    let out = &mut BufWriter::new(stdout());

    let tc: usize = scan.next();
    for _ in 0..tc {}
}

struct Scanner<R> {
    _reader: R,
    buffer: Vec<String>,
}

impl<R> Scanner<R>
where
    R: BufRead,
{
    fn new(mut reader: R) -> Self {
        let mut input = String::new();
        reader.read_to_string(&mut input).expect("Failed to read");

        let mut buffer = Vec::new();
        buffer.extend(input.split_whitespace().rev().map(String::from));

        Self {
            _reader: reader,
            buffer,
        }
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next<T>(&mut self) -> T
    where
        T: std::str::FromStr,
    {
        match self.buffer.pop() {
            Some(token) => token.parse().ok().expect("Failed to parse"),
            None => panic!("Input is all drained"),
        }
    }

    #[allow(dead_code)]
    #[inline(always)]
    fn next_option<T>(&mut self) -> Option<T>
    where
        T: std::str::FromStr,
    {
        match self.buffer.pop() {
            Some(token) => token.parse().ok(),
            None => None,
        }
    }
}
```