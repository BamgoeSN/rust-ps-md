# Arbitrary-Precision Integer

## Example
```rust,noplayground
use bigint::*;

let _a = Int::from(0i8);
let b = Int::from(4i16);
let c = Int::from(11i32);

let mut x = Int::from(30i64);
x *= &b;
println!("{}", x);

let mut y = Int::from_str("123456789123456789123456789123456789").unwrap();
let z = &y * &c;
println!("{}", y);
```

## Code
```rust,noplayground
mod bigint {
    use core::{
        fmt::Display,
        num::ParseIntError,
        ops::{Add, AddAssign, Mul, MulAssign, Sub, SubAssign},
        str::FromStr,
    };

    const CHUNK: usize = 5;
    const TENS: i64 = 100000;

    #[derive(Clone, Default, Debug, PartialEq, Eq)]
    pub struct Uint(Vec<i64>);

    macro_rules! flatten {
        ($uint:expr) => {
            let mut carry: i64 = 0;
            for i in 0..$uint.0.len() {
                $uint.0[i] += carry;
                carry = $uint.0[i].div_euclid(TENS);
                $uint.0[i] -= carry * TENS;
            }
            while carry != 0 {
                $uint.0.push(carry.rem_euclid(TENS));
                carry = carry.div_euclid(TENS);
            }
            while let Some(&x) = $uint.0.last() {
                if x != 0 {
                    break;
                }
                $uint.0.pop();
            }
        };
    }

    macro_rules! impl_from_for_uint {
        ($($t:ty),*) => {
            $(
                impl From<$t> for Uint {
                    fn from(x: $t) -> Self {
                        let mut x = Self(vec![x as i64]);
                        flatten!(x);
                        x
                    }
                }
            )*
        };
    }
    impl_from_for_uint!(u8, u16, u32, u64, u128, usize);

    impl FromStr for Uint {
        type Err = ParseIntError;
        fn from_str(s: &str) -> Result<Self, Self::Err> {
            let s = s.trim_start_matches("0");
            if s.is_empty() {
                return Ok(Self(vec![]));
            }
            let mut arr: Vec<i64> = Vec::with_capacity(s.len() / CHUNK + 2);
            let mut s = s;
            while s.len() > CHUNK {
                let (l, r) = s.split_at(s.len() - CHUNK);
                arr.push(r.parse()?);
                s = l;
            }
            arr.push(s.parse()?);
            Ok(Self(arr))
        }
    }

    impl Display for Uint {
        fn fmt(&self, f: &mut core::fmt::Formatter<'_>) -> core::fmt::Result {
            write!(f, "{}", *self.0.last().unwrap_or(&0))?;
            for &v in self.0.iter().rev().skip(1) {
                write!(f, "{:0CHUNK$}", v)?;
            }
            Ok(())
        }
    }

    impl PartialOrd for Uint {
        fn partial_cmp(&self, other: &Self) -> Option<core::cmp::Ordering> {
            use core::cmp::Ordering;
            match self.0.len().cmp(&other.0.len()) {
                Ordering::Equal => {
                    for i in (0..self.0.len()).rev() {
                        let x = self.0[i].cmp(&other.0[i]);
                        if x != Ordering::Equal {
                            return Some(x);
                        }
                    }
                    Some(Ordering::Equal)
                }
                x => Some(x),
            }
        }
    }

    impl Ord for Uint {
        fn cmp(&self, other: &Self) -> core::cmp::Ordering {
            use core::cmp::Ordering;
            match self.0.len().cmp(&other.0.len()) {
                Ordering::Equal => {
                    for i in (0..self.0.len()).rev() {
                        let x = self.0[i].cmp(&other.0[i]);
                        if x != Ordering::Equal {
                            return x;
                        }
                    }
                    Ordering::Equal
                }
                x => x,
            }
        }
    }

    impl AddAssign<&Uint> for Uint {
        fn add_assign(&mut self, rhs: &Uint) {
            if self.0.len() < rhs.0.len() {
                for i in 0..self.0.len() {
                    self.0[i] += rhs.0[i];
                }
                self.0.extend_from_slice(&rhs.0[self.0.len()..]);
            } else {
                for i in 0..rhs.0.len() {
                    self.0[i] += rhs.0[i];
                }
            }

            flatten!(self);
        }
    }

    impl Add for &Uint {
        type Output = Uint;
        fn add(self, rhs: Self) -> Self::Output {
            let mut c = self.clone();
            c += rhs;
            c
        }
    }

    impl SubAssign<&Uint> for Uint {
        fn sub_assign(&mut self, rhs: &Uint) {
            // Panics if self.len() < rhs.len(): Think it as a underflow error
            for (i, &v) in rhs.0.iter().enumerate() {
                self.0[i] -= v;
            }

            flatten!(self);
        }
    }

    impl Sub for &Uint {
        type Output = Uint;
        fn sub(self, rhs: Self) -> Self::Output {
            let mut c = self.clone();
            c -= rhs;
            c
        }
    }

    const NTT_THRES: usize = 5000;
    const KARAT_THRES: usize = 30;

    impl Mul for &Uint {
        type Output = Uint;
        fn mul(self, rhs: Self) -> Self::Output {
            let max_len = self.0.len().max(rhs.0.len());
            let max_2len = polymul::ceil_pow2(max_len);

            // For performance reasons regarding vector copying, we determine whether to use
            // NTT or not here.
            let mut ans = Uint(if max_2len > NTT_THRES {
                polymul::convolute(&self.0, &rhs.0)
            } else {
                let f: Vec<i64> = self
                    .0
                    .iter()
                    .copied()
                    .chain(core::iter::repeat(0))
                    .take(max_2len)
                    .collect();

                let g: Vec<i64> = rhs
                    .0
                    .iter()
                    .copied()
                    .chain(core::iter::repeat(0))
                    .take(max_2len)
                    .collect();

                polymul::mult_2pow(&f, &g)
            });

            flatten!(ans);
            ans
        }
    }

    impl MulAssign<&Uint> for Uint {
        fn mul_assign(&mut self, rhs: &Uint) {
            let x = &*self * rhs;
            *self = x;
        }
    }

    mod polymul {
        pub fn ceil_pow2(n: usize) -> usize {
            if n == 0 {
                return 0;
            }
            let mut m = n;
            while m != m & (!m + 1) {
                m -= m & (!m + 1);
            }
            if n == m {
                n
            } else {
                m * 2
            }
        }

        pub fn mult_2pow(f: &[i64], g: &[i64]) -> Vec<i64> {
            if f.len() > super::KARAT_THRES {
                return karatsuba(f, g);
            }

            let mut ans = vec![0; 2 * f.len()];
            for (i, &a) in f.iter().enumerate() {
                for (j, &b) in g.iter().enumerate() {
                    ans[i + j] += a * b;
                }
            }

            ans
        }

        // Length of f = Length of g = 2n = 2^(k+1)
        fn karatsuba(f: &[i64], g: &[i64]) -> Vec<i64> {
            if f.len() == 1 {
                return vec![f[0] * g[0]];
            }
            let n = f.len() / 2;
            let k = n.trailing_zeros();
            debug_assert_eq!(n, 1 << k);

            let (fl, fr) = (&f[..n], &f[n..]);
            let (gl, gr) = (&g[..n], &g[n..]);

            let flgl = mult_2pow(fl, gl);
            let frgr = mult_2pow(fr, gr);

            let fsum: Vec<_> = fl.iter().zip(fr.iter()).map(|(&a, &b)| (a + b)).collect();
            let gsum: Vec<_> = gl.iter().zip(gr.iter()).map(|(&a, &b)| (a + b)).collect();
            let fsgs = mult_2pow(&fsum, &gsum);

            let mut ans: Vec<_> = flgl.iter().copied().chain(frgr.iter().copied()).collect();
            for i in 0..fsgs.len() {
                ans[i + n] += fsgs[i];
            }
            for (i, v) in flgl
                .iter()
                .zip(frgr.iter())
                .map(|(&a, &b)| (a + b))
                .enumerate()
            {
                ans[i + n] -= v;
            }

            ans
        }

        const P2INV: i64 = 253522377;

        pub fn convolute(a: &[i64], b: &[i64]) -> Vec<i64> {
            let c1 = ntt1::convolute(a, b);
            let c2 = ntt2::convolute(a, b);

            c1.into_iter()
                .zip(c2.into_iter())
                .map(|(a1, a2)| {
                    let j = ((a1 + ntt1::NTT_P as i64 - a2) * P2INV) % ntt1::NTT_P as i64;
                    ntt2::NTT_P as i64 * j + a2
                })
                .collect()
        }

        // FFT_constname convention following https://algoshitpo.github.io/2020/05/20/fft-ntt/
        macro_rules! impl_ntt {
            ($modname:ident, $nttp:expr, $ntta:expr, $nttb:expr, $nttw:expr) => {
                mod $modname {
                    pub const NTT_P: u64 = $nttp;
                    const NTT_A: u64 = $ntta;
                    const NTT_B: u32 = $nttb;
                    const NTT_W: u64 = $nttw;

                    fn ceil_pow2(n: usize) -> usize {
                        let mut x: usize = 0;
                        while (1 << x) < n {
                            x += 1;
                        }
                        x
                    }

                    pub fn convolute(a: &[i64], b: &[i64]) -> Vec<i64> {
                        let nlen = 1 << ceil_pow2(a.len() + b.len());
                        let mut arr = vec![0; nlen];
                        let mut brr = vec![0; nlen];
                        for (i, &a) in a.iter().enumerate() {
                            arr[i] = a as u64;
                        }
                        for (i, &b) in b.iter().enumerate() {
                            brr[i] = b as u64;
                        }

                        inplace_ntt(&mut arr);
                        inplace_ntt(&mut brr);
                        let mut crr: Vec<_> =
                            arr.iter().zip(brr.iter()).map(|(&a, &b)| a * b).collect();
                        inplace_intt(&mut crr);
                        crr.iter().map(|&x| x as i64).collect()
                    }

                    #[inline(always)]
                    fn rem_pow(mut base: u64, exp: u64) -> u64 {
                        let mut result = 1u64;
                        for exp in core::iter::successors(Some(exp), |x| Some(x >> 1))
                            .take_while(|&v| v != 0)
                        {
                            if exp & 1 != 0 {
                                result *= base;
                                result %= NTT_P;
                            }
                            base *= base;
                            base %= NTT_P;
                        }
                        result
                    }

                    // unity(n, 1) ** (1<<n) = 1
                    fn unity(n: u32, k: u64) -> u64 {
                        rem_pow(rem_pow(NTT_W, NTT_A), k << (NTT_B - n))
                    }

                    fn recip(x: u64) -> u64 {
                        rem_pow(x, NTT_P - 2)
                    }

                    // Reverses k trailing bits of n
                    fn reverse_trailing_bits(n: usize, k: u32) -> usize {
                        let mut r: usize = 0;
                        for i in 0..k {
                            r |= ((n >> i) & 1) << (k - i - 1);
                        }
                        r
                    }

                    fn inplace_ntt(arr: &mut [u64]) {
                        let n: usize = arr.len();
                        let k = n.trailing_zeros();
                        assert_eq!(n, 1 << k);

                        for i in 0..n {
                            let j = reverse_trailing_bits(i, k);
                            if i < j {
                                arr.swap(i, j);
                            }
                        }

                        for x in 0..k {
                            let base: u64 = unity(x + 1, 1);
                            let s = 1 << x;
                            for i in (0..n).step_by(s << 1) {
                                let mut mult: u64 = 1;
                                for j in 0..s {
                                    let tmp = (arr[i + j + s] * mult) % NTT_P;
                                    arr[i + j + s] = (arr[i + j] + NTT_P - tmp) % NTT_P;
                                    arr[i + j] = (arr[i + j] + tmp) % NTT_P;
                                    mult *= base;
                                    mult %= NTT_P;
                                }
                            }
                        }
                    }

                    fn inplace_intt(arr: &mut [u64]) {
                        let n: usize = arr.len();
                        let k = n.trailing_zeros();
                        assert_eq!(n, 1 << k);

                        for i in 0..n {
                            let j = reverse_trailing_bits(i, k);
                            if i < j {
                                arr.swap(i, j);
                            }
                        }

                        for x in 0..k {
                            let base: u64 = recip(unity(x + 1, 1));
                            let s = 1 << x;
                            for i in (0..n).step_by(s << 1) {
                                let mut mult: u64 = 1;
                                for j in 0..s {
                                    let tmp = (arr[i + j + s] * mult) % NTT_P;
                                    arr[i + j + s] = (arr[i + j] + NTT_P - tmp) % NTT_P;
                                    arr[i + j] = (arr[i + j] + tmp) % NTT_P;
                                    mult *= base;
                                    mult %= NTT_P;
                                }
                            }
                        }

                        let r = recip(n as u64);
                        for f in arr.iter_mut() {
                            *f *= r;
                            *f %= NTT_P;
                        }
                    }
                }
            };
        }

        impl_ntt!(ntt1, 2281701377, 17, 27, 3);
        impl_ntt!(ntt2, 998244353, 119, 23, 3);
    }

    #[derive(Clone, Copy, PartialEq, Eq, Debug)]
    enum Sign {
        Neg,
        Pos, // Includes 0
    }
    use Sign::*;

    #[derive(Clone, Debug, PartialEq, Eq)]
    pub struct Int {
        sign: Sign,
        nat: Uint,
    }

    macro_rules! impl_from_for_int {
        ($($u:ty, $s:ty);*) => {
            $(
                impl From<$u> for Int {
                    fn from(x: $u) -> Self {
                        Self { sign: Pos, nat: x.into() }
                    }
                }
                impl From<$s> for Int {
                    fn from(x: $s) -> Self {
                        if x < 0 {
                            Self { sign: Neg, nat: ((-x) as $u).into() }
                        } else {
                            Self { sign: Pos, nat: (x as $u).into() }
                        }
                    }
                }
            )*
        };
    }

    impl_from_for_int!(u8, i8; u16, i16; u32, i32; u64, i64; u128, i128; usize, isize);

    impl FromStr for Int {
        type Err = ParseIntError;
        fn from_str(s: &str) -> Result<Self, Self::Err> {
            if s.len() == 0 {
                panic!("Empty string - TODO: Add a proper error propagation");
            }
            let mut x = match s.strip_prefix("-") {
                Some(t) => Self {
                    sign: Neg,
                    nat: t.parse()?,
                },
                None => Self {
                    sign: Pos,
                    nat: s.parse()?,
                },
            };
            if x.sign == Neg && x.nat.0.len() == 0 {
                x.sign = Pos;
            }
            Ok(x)
        }
    }

    impl Display for Int {
        fn fmt(&self, f: &mut core::fmt::Formatter<'_>) -> core::fmt::Result {
            if let Neg = self.sign {
                write!(f, "-")?;
            }
            write!(f, "{}", self.nat)
        }
    }

    impl PartialOrd for Int {
        fn partial_cmp(&self, other: &Self) -> Option<core::cmp::Ordering> {
            use core::cmp::Ordering;
            match (self.sign, other.sign) {
                (Neg, Neg) => other.nat.partial_cmp(&self.nat),
                (Neg, Pos) => Some(Ordering::Less),
                (Pos, Neg) => Some(Ordering::Greater),
                (Pos, Pos) => self.nat.partial_cmp(&other.nat),
            }
        }
    }

    impl Ord for Int {
        fn cmp(&self, other: &Self) -> core::cmp::Ordering {
            use core::cmp::Ordering;
            match (self.sign, other.sign) {
                (Neg, Neg) => other.nat.cmp(&self.nat),
                (Neg, Pos) => Ordering::Less,
                (Pos, Neg) => Ordering::Greater,
                (Pos, Pos) => self.nat.cmp(&other.nat),
            }
        }
    }

    impl AddAssign<&Int> for Int {
        fn add_assign(&mut self, rhs: &Int) {
            match (self.sign, rhs.sign) {
                (Neg, Neg) => {
                    self.nat += &rhs.nat;
                }
                (Neg, Pos) => {
                    if self.nat >= rhs.nat {
                        self.nat -= &rhs.nat;
                    } else {
                        let c = &rhs.nat - &self.nat;
                        self.nat = c;
                        self.sign = Pos;
                    }
                    if self.nat.0.len() == 0 {
                        self.sign = Pos;
                    }
                }
                (Pos, Neg) => {
                    if self.nat >= rhs.nat {
                        self.nat -= &rhs.nat;
                    } else {
                        let c = &rhs.nat - &self.nat;
                        self.nat = c;
                        self.sign = Neg;
                    }
                    if self.nat.0.len() == 0 {
                        self.sign = Pos;
                    }
                }
                (Pos, Pos) => {
                    self.nat += &rhs.nat;
                }
            }
        }
    }

    impl Add for &Int {
        type Output = Int;
        fn add(self, rhs: Self) -> Self::Output {
            let mut ans = self.clone();
            ans += rhs;
            ans
        }
    }

    impl SubAssign<&Int> for Int {
        fn sub_assign(&mut self, rhs: &Int) {
            match (self.sign, rhs.sign) {
                (Neg, Pos) => {
                    self.nat += &rhs.nat;
                }
                (Neg, Neg) => {
                    if self.nat >= rhs.nat {
                        self.nat -= &rhs.nat;
                    } else {
                        let c = &rhs.nat - &self.nat;
                        self.nat = c;
                        self.sign = Pos;
                    }
                    if self.nat.0.len() == 0 {
                        self.sign = Pos;
                    }
                }
                (Pos, Pos) => {
                    if self.nat >= rhs.nat {
                        self.nat -= &rhs.nat;
                    } else {
                        let c = &rhs.nat - &self.nat;
                        self.nat = c;
                        self.sign = Neg;
                    }
                    if self.nat.0.len() == 0 {
                        self.sign = Pos;
                    }
                }
                (Pos, Neg) => {
                    self.nat += &rhs.nat;
                }
            }
        }
    }

    impl Sub for &Int {
        type Output = Int;
        fn sub(self, rhs: Self) -> Self::Output {
            let mut ans = self.clone();
            ans -= &rhs;
            ans
        }
    }

    impl Mul for &Int {
        type Output = Int;
        fn mul(self, rhs: Self) -> Self::Output {
            let x = &self.nat * &rhs.nat;
            if x.0.len() == 0 || self.sign == rhs.sign {
                Int { sign: Pos, nat: x }
            } else {
                Int { sign: Neg, nat: x }
            }
        }
    }

    impl MulAssign<&Int> for Int {
        fn mul_assign(&mut self, rhs: &Int) {
            let x = &self.nat * &rhs.nat;
            self.nat = x;
            if self.nat.0.len() == 0 || self.sign == rhs.sign {
                self.sign = Pos;
            } else {
                self.sign = Neg;
            }
        }
    }
}
```