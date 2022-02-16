# Base Template
## w/o Testcases
```rust,noplayground
use std::io::*;

use fastio::*;

fn main() {
    let stdin = std::fs::File::open("input.txt").ok().unwrap();
    let handle = BufReader::new(stdin);
    // let stdin = stdin();
    // let handle = stdin.lock();
    let mut scan = Scanner::new(handle);
    let out = &mut BufWriter::new(stdout());
    
}

mod fastio {
    use std::io::BufRead;

    pub struct Scanner<R> {
        reader: R,
        buffer: Vec<String>,
    }

    impl<R> Scanner<R> {
        pub fn new(reader: R) -> Self {
            Scanner {
                reader,
                buffer: Vec::new(),
            }
        }
    }

    impl<R> Scanner<R>
    where
        R: BufRead,
    {
        pub fn next<T>(&mut self) -> T
        where
            T: std::str::FromStr,
        {
            loop {
                if let Some(token) = self.buffer.pop() {
                    return token.parse().ok().expect("Failed to parse");
                }
                let mut input = String::new();
                self.reader.read_line(&mut input).expect("Failed to read");
                self.buffer
                    .extend(input.split_whitespace().rev().map(String::from));
            }
        }
    }
}
```

## w/ Testcases
```rust,noplayground
use std::io::*;

use fastio::*;

fn main() {
    let stdin = std::fs::File::open("input.txt").ok().unwrap();
    let handle = BufReader::new(stdin);
    // let stdin = stdin();
    // let handle = stdin.lock();
    let mut scan = Scanner::new(handle);
    let out = &mut BufWriter::new(stdout());

    let tc: usize = scan.next();
    for _ in 0..tc {
        
    }
}

mod fastio {
    use std::io::BufRead;

    pub struct Scanner<R> {
        reader: R,
        buffer: Vec<String>,
    }

    impl<R> Scanner<R> {
        pub fn new(reader: R) -> Self {
            Scanner {
                reader,
                buffer: Vec::new(),
            }
        }
    }

    impl<R> Scanner<R>
    where
        R: BufRead,
    {
        pub fn next<T>(&mut self) -> T
        where
            T: std::str::FromStr,
        {
            loop {
                if let Some(token) = self.buffer.pop() {
                    return token.parse().ok().expect("Failed to parse");
                }
                let mut input = String::new();
                self.reader.read_line(&mut input).expect("Failed to read");
                self.buffer
                    .extend(input.split_whitespace().rev().map(String::from));
            }
        }
    }
}
```