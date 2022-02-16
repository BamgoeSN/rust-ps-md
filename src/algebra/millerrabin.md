# Miller-Rabin Primality Test

Deterministic Miller-Rabin primality test determines whether a certain unsigned integer is a prime or not within \\(O(\log{n})\\) time. This test only works for integers under \\(2^{64}\\).
```rust,noplayground
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
```

## Example
```rust,noplayground
assert!(3284729387909u64.is_prime());
assert!(!3284729387911u64.is_prime()); // 53×61976026187
```