# Fast IO

- `get_input() -> &'static str` reads every input from stdin, leaks it and returns it as `&'static str`.
  The function leaks the string because it's easier to handle input in CP this way.

- `Tokenizer` tokenizes a string based on a function its fed when initialized.

  - The default template initializes a tokenizer `sc` with `Tokenizer::new(input_str, |s| s.split_ascii_whitespace())`.
    This means that `sc` tokenizes the input by ascii whitespace.
    If you want to tokenize the input with non-ascii whitespace as well, then change `split_ascii_whitespace` to `split_whitespace`.
    If you want a tokenizer which splits a string by lines, then change `split_ascii_whitespace` to `lines`.

  - `Tokenizer::next(&mut Self) -> T` parses next token into `T` and returns it. If there's no tokens left or it fails to parse, the program panics.

  - `Tokenizer::next_str(&mut self) -> &str` returns next token.

  - `Tokenizer::next_ok(&mut self) -> Result<T, InputError>` parses the next token into `T`. If there's no tokens left in the string, it returns `Err(InputExhaust)`.
    If it fails to parse, it returns `Err(ParseError(token))`. Otherwise, it returns the parsed `T` value.

  - `Tokenizer::next_str_ok(&mut self) -> Result<T, InputError>` returns next token if there's any. Otherwise, it returns `Err(InputExhaust)`.

  - `Tokenizer::next_iter(&mut self) -> impl Iterator<Item = T>` returns an iterator repeatedly consumes and parses tokens into `T`.
    If `Tokenizer` meets a token that can't be parsed into `T`, it just skips to the next token until it can parse it into `T`.
    The iterator ends only when the entire tokens are consumed. This behavior can be used for reading up to EOF, or blocked by `take` method for reading given number of tokens.

## Example

```rust,noplayground
let n: usize = sc.next();
let arr: Vec<u64> = sc.next_iter().take(m).collect();
let (m, k): (u32, u64) = sc.next();
let text = sc.next_str();
outln!("{:?}", [m, k]);

while let Ok(n @ ..1) = sc.next_ok::<u32>() {
    println!("{}", n);
    if sc.next_str_ok().is_none() {
        outln!("EOF");
    }
}
```

## Code

### For Recent Rust versions (Almost every OJs except AtCoder)

```rust,noplayground
#![no_main]

#[no_mangle]
fn main() -> i32 {
    // FastIO
    use fastio::*;
    let input_str = get_input();
    let mut sc = Tokenizer::new(input_str, |s| s.split_ascii_whitespace());
    use std::io::{stdout, BufWriter, Write};
    let stdout = stdout();
    let wr = &mut BufWriter::new(stdout.lock());

    // FastIO Macros
    macro_rules! out { ($($arg:tt)*) => { write!(wr, $($arg)*).ok(); }; }
    macro_rules! outln { ($($arg:tt)*) => { writeln!(wr, $($arg)*).ok(); }; }

    // Main

    wr.flush().unwrap();
    0
}

#[allow(unused)]
mod fastio {
    use std::{fmt, io, num::*, slice::*, str::*};

    #[link(name = "c")]
    extern "C" {}

    pub fn get_input() -> &'static str {
        let buf = io::read_to_string(io::stdin()).unwrap();
        Box::leak(buf.into_boxed_str())
    }

    pub enum InputError<'t> {
        InputExhaust,
        ParseError(&'t str),
    }
    use InputError::*;

    impl<'t> fmt::Debug for InputError<'t> {
        fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
            match self {
                InputExhaust => f.debug_struct("InputExhaust").finish(),
                ParseError(s) => f.debug_struct("ParseError").field("str", s).finish(),
            }
        }
    }

    pub trait Atom: Sized {
        fn parse_from(s: &str) -> Result<Self, InputError>;
    }

    pub trait IterParse: Sized {
        fn parse_from<'s, 't: 's, It>(it: &'s mut It) -> Result<Self, InputError<'t>>
        where
            It: Iterator<Item = &'t str>;
    }

    macro_rules! impl_trait_for_fromstr {
        ($($t:ty) *) => { $(
            impl Atom for $t { fn parse_from(s: &str) -> Result<Self, InputError> { s.parse().map_err(|_| ParseError(s)) } }
            impl IterParse for $t {
                fn parse_from<'s, 't: 's, It>(it: &'s mut It) -> Result<Self, InputError<'t>> where It: Iterator<Item = &'t str> {
                    it.next().map_or( Err(InputExhaust), <Self as Atom>::parse_from )
                }
            }
        )* };
    }

    impl_trait_for_fromstr!(bool char String);
    impl_trait_for_fromstr!(f32 f64 i8 i16 i32 i64 i128 isize u8 u16 u32 u64 u128 usize);
    impl_trait_for_fromstr!(NonZeroI8 NonZeroI16 NonZeroI32 NonZeroI64 NonZeroI128 NonZeroIsize);
    impl_trait_for_fromstr!(NonZeroU8 NonZeroU16 NonZeroU32 NonZeroU64 NonZeroU128 NonZeroUsize);

    macro_rules! impl_iterparse_for_tuple {
        ($($t:ident) *) => {
            impl<$($t),*> IterParse for ($($t),*) where $($t: IterParse),* {
                fn parse_from<'s, 't: 's, It>(it: &'s mut It) -> Result<Self, InputError<'t>> where It: Iterator<Item = &'t str> {
                    Ok(( $($t::parse_from(it)?),* ))
                }
            }
        };
    }

    impl_iterparse_for_tuple!();
    impl_iterparse_for_tuple!(A B);
    impl_iterparse_for_tuple!(A B C);
    impl_iterparse_for_tuple!(A B C D);
    impl_iterparse_for_tuple!(A B C D E);
    impl_iterparse_for_tuple!(A B C D E F);
    impl_iterparse_for_tuple!(A B C D E F G);
    impl_iterparse_for_tuple!(A B C D E F G H);
    impl_iterparse_for_tuple!(A B C D E F G H I);
    impl_iterparse_for_tuple!(A B C D E F G H I J);
    impl_iterparse_for_tuple!(A B C D E F G H I J K);
    impl_iterparse_for_tuple!(A B C D E F G H I J K L);
    impl_iterparse_for_tuple!(A B C D E F G H I J K L M);

    pub struct Tokenizer<It> {
        it: It,
    }

    impl<'arg, 'str: 'arg, It> Tokenizer<It> {
        pub fn new(s: &'str str, split: impl FnOnce(&'arg str) -> It) -> Self {
            Self { it: split(s) }
        }
    }

    impl<'t, It> Tokenizer<It>
    where
        It: Iterator<Item = &'t str>,
    {
        pub fn next<T: IterParse>(&mut self) -> T {
            T::parse_from(&mut self.it).unwrap()
        }
        pub fn next_str(&mut self) -> &'t str {
            self.it.next().unwrap()
        }
        pub fn next_ok<T: IterParse>(&mut self) -> Result<T, InputError<'t>> {
            T::parse_from(&mut self.it)
        }
        pub fn next_str_ok(&mut self) -> Option<&'t str> {
            self.it.next()
        }
        pub fn next_iter<T: IterParse>(&mut self) -> impl Iterator<Item = T> + '_ {
            std::iter::repeat_with(move || self.next_ok().ok()).map_while(|x| x)
        }
    }
}
```

### For Older Versions of Rust (AtCoder)

```rust,noplayground
#![no_main]

#[no_mangle]
fn main() -> i32 {
    // FastIO
    use fastio::*;
    let input_str = get_input();
    let mut sc = Tokenizer::new(input_str, |s| s.split_ascii_whitespace());
    use std::io::{stdout, BufWriter, Write};
    let stdout = stdout();
    let wr = &mut BufWriter::new(stdout.lock());

    // FastIO Macros
    macro_rules! out { ($($arg:tt)*) => { write!(wr, $($arg)*).ok(); }; }
    macro_rules! outln { ($($arg:tt)*) => { writeln!(wr, $($arg)*).ok(); }; }

    // Main

    wr.flush().unwrap();
    0
}

#[allow(unused)]
mod fastio {
    use std::{fmt, io, num::*, slice::*, str::*};

    #[link(name = "c")]
    extern "C" {}

    pub fn get_input() -> &'static str {
        use io::Read;
        let mut buf = String::new();
        io::stdin().read_to_string(&mut buf);
        Box::leak(buf.into_boxed_str())
    }

    pub enum InputError<'t> {
        InputExhaust,
        ParseError(&'t str),
    }
    use InputError::*;

    impl<'t> fmt::Debug for InputError<'t> {
        fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
            match self {
                InputExhaust => f.debug_struct("InputExhaust").finish(),
                ParseError(s) => f.debug_struct("ParseError").field("str", s).finish(),
            }
        }
    }

    pub trait Atom: Sized {
        fn parse_from(s: &str) -> Result<Self, InputError>;
    }

    pub trait IterParse: Sized {
        fn parse_from<'s, 't: 's, It>(it: &'s mut It) -> Result<Self, InputError<'t>>
        where
            It: Iterator<Item = &'t str>;
    }

    macro_rules! impl_trait_for_fromstr {
        ($($t:ty) *) => { $(
            impl Atom for $t { fn parse_from(s: &str) -> Result<Self, InputError> { s.parse().map_err(|_| ParseError(s)) } }
            impl IterParse for $t {
                fn parse_from<'s, 't: 's, It>(it: &'s mut It) -> Result<Self, InputError<'t>> where It: Iterator<Item = &'t str> {
                    it.next().map_or( Err(InputExhaust), <Self as Atom>::parse_from )
                }
            }
        )* };
    }

    impl_trait_for_fromstr!(bool char String);
    impl_trait_for_fromstr!(f32 f64 i8 i16 i32 i64 i128 isize u8 u16 u32 u64 u128 usize);
    impl_trait_for_fromstr!(NonZeroI8 NonZeroI16 NonZeroI32 NonZeroI64 NonZeroI128 NonZeroIsize);
    impl_trait_for_fromstr!(NonZeroU8 NonZeroU16 NonZeroU32 NonZeroU64 NonZeroU128 NonZeroUsize);

    macro_rules! impl_iterparse_for_tuple {
        ($($t:ident) *) => {
            impl<$($t),*> IterParse for ($($t),*) where $($t: IterParse),* {
                fn parse_from<'s, 't: 's, It>(it: &'s mut It) -> Result<Self, InputError<'t>> where It: Iterator<Item = &'t str> {
                    Ok(( $($t::parse_from(it)?),* ))
                }
            }
        };
    }

    impl_iterparse_for_tuple!();
    impl_iterparse_for_tuple!(A B);
    impl_iterparse_for_tuple!(A B C);
    impl_iterparse_for_tuple!(A B C D);
    impl_iterparse_for_tuple!(A B C D E);
    impl_iterparse_for_tuple!(A B C D E F);
    impl_iterparse_for_tuple!(A B C D E F G);
    impl_iterparse_for_tuple!(A B C D E F G H);
    impl_iterparse_for_tuple!(A B C D E F G H I);
    impl_iterparse_for_tuple!(A B C D E F G H I J);
    impl_iterparse_for_tuple!(A B C D E F G H I J K);
    impl_iterparse_for_tuple!(A B C D E F G H I J K L);
    impl_iterparse_for_tuple!(A B C D E F G H I J K L M);

    pub struct Tokenizer<It> {
        it: It,
    }

    impl<'arg, 'str: 'arg, It> Tokenizer<It> {
        pub fn new(s: &'str str, split: impl FnOnce(&'arg str) -> It) -> Self {
            Self { it: split(s) }
        }
    }

    impl<'t, It> Tokenizer<It>
    where
        It: Iterator<Item = &'t str>,
    {
        pub fn next<T: IterParse>(&mut self) -> T {
            T::parse_from(&mut self.it).unwrap()
        }
        pub fn next_str(&mut self) -> &'t str {
            self.it.next().unwrap()
        }
        pub fn next_ok<T: IterParse>(&mut self) -> Result<T, InputError<'t>> {
            T::parse_from(&mut self.it)
        }
        pub fn next_str_ok(&mut self) -> Option<&'t str> {
            self.it.next()
        }
        pub fn next_iter<'s, T: IterParse>(&'s mut self) -> impl Iterator<Item = T> + '_
        where
            't: 's,
        {
            std::iter::repeat_with(move || self.next_ok())
                .take_while(|x| x.is_ok())
                .flatten()
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
