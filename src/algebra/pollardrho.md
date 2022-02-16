# Pollard Rho Algorithm
Pollard rho algorithm is a randomized algorithm which factorizes a number in an average of \\(O(n^{1/4})\\) time.
```rust,noplayground
pub struct RNG {
    val: u64,
}

impl RNG {
    /// Returns a new RNG instance starting with a given seed.
    pub fn new(seed: u64) -> Self {
        Self { val: seed }
    }

    /// Returns a random u64 number.
    pub fn next(&mut self) -> u64 {
        let mut x = self.val;
        x ^= x << 13;
        x ^= x >> 7;
        x ^= x << 17;
        self.val = x;
        x
    }

    /// Returns a random f64 number within [0, 1].
    pub fn next_f64(&mut self) -> f64 {
        (self.next() as f64) / (u64::MAX as f64)
    }

    /// Returns a random u64 number within [l, r).
    pub fn range_u64(&mut self, l: u64, r: u64) -> u64 {
        l + self.next() % (r - l)
    }

    /// Returns a random i64 number within [l, r).
    pub fn range_i64(&mut self, l: i64, r: i64) -> i64 {
        l + (self.next() / 2) as i64 % (r - l)
    }

    /// Returns a random f64 number within [l, r].
    pub fn range_f64(&mut self, l: f64, r: f64) -> f64 {
        l + self.next_f64() * (r - l)
    }
}

pub trait Miller {
    fn rem_mul(x: Self, y: Self, m: Self) -> Self;
    fn rem_pow(base: Self, exp: Self, m: Self) -> Self;
    fn naive_primality(self) -> bool;
    fn miller_rabin(self, a: Self) -> bool;
    fn miller_primality(self) -> bool;
    fn is_prime(self) -> bool;
}

macro_rules! miller_rabin_impl {
    ($t:ty, $u:ty, $limit:expr,  $($x:expr),*) => {
        impl Miller for $t {
            #[inline]
            fn rem_mul(x: $t, y: $t, m: $t) -> $t {
                ((x as $u * y as $u) % m as $u) as $t
            }
            fn rem_pow(mut base: $t, mut exp: $t, m: $t) -> $t {
                exp %= m;
                let mut r: $t = 1;
                while exp != 0 {
                    if exp & 1 != 0 {
                        r = Self::rem_mul(r, base, m);
                    }
                    exp >>= 1;
                    base = Self::rem_mul(base, base, m);
                }
                r
            }
            fn naive_primality(self) -> bool {
                let mut i: $t = 2;
                while i * i <= self {
                    if self % i == 0 {
                        return false;
                    }
                    i += 1;
                }
                true
            }
            fn miller_rabin(self, a: $t) -> bool {
                let d = self - 1;
                let mut p = d;
                while p&1==0 {
                    p>>=1;
                }
                let mut t = Self::rem_pow(a, p, self);
                let at_last = t == d || t == 1;

                while p!=d {
                    p <<= 1;
                    t = Self::rem_mul(t, t, self);
                    if t == self-1 {return true;}
                }

                at_last
            }
            fn miller_primality(self) -> bool {
                $(
                    if !self.miller_rabin($x) {
                        return false;
                    }
                )*
                true
            }
            fn is_prime(self) -> bool {
                if self <= 1 {
                    false
                } else if self <= ($limit) {
                    Self::naive_primality(self)
                } else {
                    Self::miller_primality(self)
                }
            }
        }
    };
}

miller_rabin_impl!(u8, u16, 254, 2);
miller_rabin_impl!(u16, u32, 2000, 2, 3);
miller_rabin_impl!(u32, u64, 10000, 2, 7, 61);
miller_rabin_impl!(u64, u128, 1000000, 2, 325, 9375, 28178, 450775, 9780504, 1795265022);

fn gcd(mut a: u64, mut b: u64) -> u64 {
    while b != 0 {
        let t = b;
        b = a % b;
        a = t;
    }
    a
}

#[inline]
fn rem_add(a: u64, b: u64, m: u64) -> u64 {
    ((a as u128 + b as u128) % m as u128) as u64
}

#[inline]
fn abs_diff(a: u64, b: u64) -> u64 {
    if a > b {
        a - b
    } else {
        b - a
    }
}

pub fn factorize(mut n: u64, rng: &mut RNG) -> Vec<u64> {
    if n <= 1 {
        return Vec::new();
    }
    let mut arr: Vec<u64> = Vec::new();
    while n & 1 == 0 {
        n >>= 1;
        arr.push(2);
    }
    rho(n, &mut arr, rng);
    arr
}

fn rho(n: u64, arr: &mut Vec<u64>, rng: &mut RNG) {
    if n <= 1 {
        return;
    } else if n.is_prime() {
        arr.push(n);
        return;
    }

    let mut i = 0u64;
    let mut x = rng.next() % n;
    let mut y = x;
    let mut k = 2u64;
    let mut d;
    let mut reset_limit = 500000;

    loop {
        i += 1;
        x = rem_add(Miller::rem_mul(x, x, n), n - 1, n);
        d = gcd(abs_diff(y, x), n);
        if d == n || i >= reset_limit {
            // Reset
            reset_limit = reset_limit * 3 / 2;
            i = 0;
            x = rng.next();
            y = x;
        }
        if d != 1 {
            break;
        }
        if i == k {
            y = x;
            k <<= 1;
        }
    }

    if d != n {
        rho(d, arr, rng);
        rho(n / d, arr, rng);
        return;
    }

    i = 3;
    while i * i <= n {
        if n % i == 0 {
            rho(i, arr, rng);
            rho(d / i, arr, rng);
            return;
        }
        i += 2;
    }
}
```

## Example
```rust,noplayground
let mut rng = RNG::new(12345);
let a = 3284729387911u64;
let mut factors = factorize(a, &mut rng);
factors.sort();
assert_eq!(factors, [53, 61976026187]);
```