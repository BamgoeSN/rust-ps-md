# Fast IO

`get_input()` returns every text read from the standard input, using `mmap`. Because of this, the input must be read by file redirection as `cargo run --release < input.txt`. THIS FUNCTION DOESN'T WORK ON WINDOWS. THE FUNCTION MUST BE REWRITTEN MANUALLY TO BE USED FOR <u>**CODEFORCES**</u>, WHICH RUNS CODES ON WINDOWS.

`Scanner` is a structure which can read either lines or tokens out of a string. `Scanner::tokenize(str)` tokenizes `str` by whitespaces, such as ` ` spaces, `\t` tabs, and `\n` newlines.

`sc` in `main()` function is a scanner instance, which splits `input_str` by whitespaces. `sc.next_str()` returns the next token as `&str`. If the input has been exhausted, the program panics. `sc.next::<T>()` reads the next token, parses it into a type `T`, and returns it. If the input has been exhausted or the parse fails, the program panics.

Both function has its "Option" equivalents. `sc.next_str_option()` returns the next token as `Some(&str)`. This function returns `None` instead of panicing the whole program, compared to `sc.next_str()`. `sc.next_option()` works just like `sc.next()` except that it returns `Option<T>` so that the function returns `None` instead of panicing. These functions are useful when the code needs to detect `EOF`.

`next!` macros are for easy-typing for inputs. `next!()` is equivalent to `sc.next()`. You can specify the return type of it by using `next!(T)`. Multiple types can also be used as arguments, delimited by spaces: `next!(T U V)` returns a tuple `(T, U, V)`. Lastly, `next!(str)` is equivalent to `sc.next_str()`.

`out!` and `outln!` are just the same with `print!` and `println!`, but the stdout flush is buffered for faster output.

### Example

```rust,noplayground
// Main
let n: usize = next!();      // Identical to `let n: usize = sc.next();`
let m = next!(usize);        // Identical to `let m = sc.next::<usize>();`
let (a, b) = next!(u32 i64); // Identical to `let (a, b): (u32, i64) = (sc.next(), sc.next());`
let arr: Vec<u64> = (0..n).map(|_| next!()).collect();

let word = next!(str);

out!("{} ", a);
outln!("{}", b);
outln!("{:?}", [n, m]);
```

## Code

### For Linux (Almost every OJs including AtCoder)

```rust,noplayground
#![no_main]

#[no_mangle]
fn main() -> i32 {
    // FastIO
    use fastio::*;
    let input_str = get_input();
    let mut sc: Splitter<_> = Splitter::new(input_str, |s| s.split_ascii_whitespace());
    use std::io::*;
    let stdout = stdout();
    let wr = &mut BufWriter::new(stdout.lock());

    // FastIO Macros
    macro_rules! next {
        () => { sc.next() };
        ($($t:ty) +) => { ($(sc.next::<$t>()),+) };
    }
    macro_rules! out { ($($arg:tt)*) => { write!(wr, $($arg)*).ok(); }; }
    macro_rules! outln { ($($arg:tt)*) => { writeln!(wr, $($arg)*).ok(); }; }

    // Main

    wr.flush().unwrap();
    0
}

mod fastio {
    use core::{slice::*, str::*};

    #[link(name = "c")]
    extern "C" {
        fn mmap(addr: usize, len: usize, p: i32, f: i32, fd: i32, o: i64) -> *mut u8;
        fn fstat(fd: i32, stat: *mut usize) -> i32;
    }

    pub fn get_input() -> &'static str {
        let mut stat = [0; 20];
        unsafe { fstat(0, stat.as_mut_ptr()) };
        let buffer = unsafe { mmap(0, stat[6], 1, 2, 0, 0) };
        unsafe { from_utf8_unchecked(from_raw_parts(buffer, stat[6])) }
    }

    pub struct Splitter<I: Iterator> {
        it: I,
    }

    impl<'a, 'b: 'a, T: Iterator> Splitter<T> {
        pub fn new(s: &'b str, split: impl FnOnce(&'a str) -> T) -> Self {
            Self { it: split(s) }
        }
    }

    impl<'a, I: Iterator<Item = &'a str>> Splitter<I> {
        pub fn next<T: FromStr>(&mut self) -> T {
            self.it.next().unwrap().parse().ok().unwrap()
        }
        pub fn next_str(&mut self) -> &'a str {
            self.it.next().unwrap()
        }
        pub fn next_opt<T: FromStr>(&mut self) -> Option<T> {
            self.it.next().and_then(|s| s.parse().ok())
        }
        pub fn next_str_opt(&mut self) -> Option<&'a str> {
            self.it.next()
        }
    }
}
```

### For Windows (Codeforces)

```rust,noplayground
#![no_main]

#[no_mangle]
fn main() -> i32 {
    // FastIO
    use fastio::*;
    let input_str = get_input();
    let mut sc: Splitter<_> = Splitter::new(&input_str, |s| s.split_ascii_whitespace());
    use std::io::*;
    let stdout = stdout();
    let wr = &mut BufWriter::new(stdout.lock());

    // FastIO Macros
    macro_rules! next {
        () => { sc.next() };
        ($($t:ty) +) => { ($(sc.next::<$t>()),+) };
    }
    macro_rules! out { ($($arg:tt)*) => { write!(wr, $($arg)*).ok(); }; }
    macro_rules! outln { ($($arg:tt)*) => { writeln!(wr, $($arg)*).ok(); }; }

    // Main

    wr.flush().unwrap();
    0
}

mod fastio {
    use std::{io::*, str::*};

    pub fn get_input() -> String {
        let mut buf = String::new();
        stdin().read_to_string(&mut buf).unwrap();
        buf
    }

    pub struct Splitter<I: Iterator> {
        it: I,
    }

    impl<'a, 'b: 'a, T: Iterator> Splitter<T> {
        pub fn new(s: &'b str, split: impl FnOnce(&'a str) -> T) -> Self {
            Self { it: split(s) }
        }
    }

    impl<'a, I: Iterator<Item = &'a str>> Splitter<I> {
        pub fn next<T: FromStr>(&mut self) -> T {
            self.it.next().unwrap().parse().ok().unwrap()
        }
        pub fn next_str(&mut self) -> &'a str {
            self.it.next().unwrap()
        }
        pub fn next_opt<T: FromStr>(&mut self) -> Option<T> {
            self.it.next().and_then(|s| s.parse().ok())
        }
        pub fn next_str_opt(&mut self) -> Option<&'a str> {
            self.it.next()
        }
    }
}
```

### With faster print (Old)

```rust,noplayground
fn main() {
    // FastIO
    use fastio::*;
    let input_str = get_input();
    let mut sc: Scanner<_> = Scanner::tokenize(input_str);
    let mut out = Flusher::with_capacity(1 << 18);

    // FastIO Macros
    macro_rules! next {
        () => { sc.next() };
        (str) => { sc.next_str() };
        ($($t:ty) +) => { ($(sc.next::<$t>()),+) };
    }
    macro_rules! out { ($($arg:tt),*) => {$( $arg.push_num(&mut out); out.buf.push(b' '); )*}; }
    macro_rules! outln { ($($arg:tt),*) => { out!($($arg),*); out.buf.push(b'\n'); } }

    // Main
    let n: usize = next!();
    outln!(n);
}

mod fastio {
    use std::io::{stdout, Write};

    extern "C" {
        fn mmap(addr: usize, len: usize, p: i32, f: i32, fd: i32, o: i64) -> *mut u8;
        fn fstat(fd: i32, stat: *mut usize) -> i32;
    }

    pub fn get_input() -> &'static str {
        let mut stat = [0; 20];
        unsafe { fstat(0, (&mut stat).as_mut_ptr()) };
        let buffer = unsafe { mmap(0, stat[6], 1, 2, 0, 0) };
        unsafe { std::str::from_utf8_unchecked(std::slice::from_raw_parts(buffer, stat[6])) }
    }

    pub struct Scanner<'a, I: Iterator<Item = &'a str>> {
        it: I,
    }

    impl<'a> Scanner<'a, std::str::SplitAsciiWhitespace<'a>> {
        pub fn tokenize(s: &'a str) -> Self {
            Self {
                it: s.split_ascii_whitespace(),
            }
        }
    }

    impl<'a> Scanner<'a, std::str::Lines<'a>> {
        pub fn lines(s: &'a str) -> Self {
            Self { it: s.lines() }
        }
    }

    impl<'a, I: Iterator<Item = &'a str>> Scanner<'a, I> {
        #[inline(always)]
        pub fn next<T: std::str::FromStr>(&mut self) -> T {
            self.it.next().unwrap().parse().ok().unwrap()
        }
        #[inline(always)]
        pub fn next_str(&mut self) -> &'a str {
            self.it.next().unwrap()
        }
        #[inline(always)]
        pub fn next_option<T: std::str::FromStr>(&mut self) -> Option<T> {
            self.it.next().and_then(|s| s.parse().ok())
        }
        #[inline(always)]
        pub fn next_str_option(&mut self) -> Option<&'a str> {
            self.it.next()
        }
    }

    pub struct Flusher {
        pub buf: Vec<u8>,
    }

    impl Flusher {
        pub fn with_capacity(cap: usize) -> Self {
            Self {
                buf: Vec::with_capacity(cap),
            }
        }
    }

    impl Drop for Flusher {
        fn drop(&mut self) {
            stdout().write_all(&self.buf).ok();
        }
    }

    pub trait PushPrint {
        fn push_num(self, wr: &mut Flusher);
    }

    impl PushPrint for &str {
        fn push_num(self, wr: &mut Flusher) {
            wr.buf.extend_from_slice(self.as_bytes());
        }
    }

    macro_rules! impl_pushprint {
        ($($i:ty;$u:ty) *) => {
            $(
                impl PushPrint for $i {
                    fn push_num(self, wr: &mut Flusher) {
                        if self < 0 {
                            wr.buf.push(b'-');
                            ((-self) as $u).push_num(wr);
                        } else {
                            (self as $u).push_num(wr);
                        }
                    }
                }

                impl PushPrint for $u {
                    fn push_num(self, wr: &mut Flusher) {
                        if self >= 10 {
                            (self / 10).push_num(wr);
                        }
                        wr.buf.push((self % 10) as u8 + b'0');
                    }
                }
            )*
        };
    }

    impl_pushprint!(i8;u8 i16;u16 i32;u32 i64;u64 i128;u128 isize;usize);
}

```
