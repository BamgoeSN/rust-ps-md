# Base Template

```rust,noplayground
fn main() {
    // FastIO
    use {fastio::*, std::io::*};
    let input_str: &str = get_input();
    let mut scan: Scanner<_> = Scanner::tokenize(input_str);
    let stdout = stdout();
    let out = &mut BufWriter::with_capacity(262144, stdout.lock());

    // FastIO Macros
    macro_rules! next {
        () => { scan.next() };
        (str) => { scan.next_str() };
        ($($t:ty) +) => { ($(scan.next::<$t>()),+) };
        ($n:expr) => { (0..$n).map(|_| next!()).collect() };
    }
    macro_rules! out { ($($arg:tt)*) => { write!(out, $($arg)*).ok(); }; }
    macro_rules! outln { ($($arg:tt)*) => { writeln!(out, $($arg)*).ok(); }; }

    // Main
}

mod fastio {
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
}
```
