# Miller-Rabin Primality Test

Deterministic Miller-Rabin primality test determines whether a certain unsigned integer is a prime or not within \\(O(\log{n})\\) time. This test only works for integers under \\(2^{64}\\). \
Required snippets: [Zero](../misc/zero_one_trait.md#Zero), [One](../misc/zero_one_trait.md#One)
```rust,noplayground
use std::ops::{BitAnd, Mul, Rem, RemAssign, ShrAssign};

trait Cast<T> {
    fn cast_to(self) -> T;
}

macro_rules! impl_primitive_cast {
    ($t:ty, $u:ty) => {
        impl Cast<$t> for $u {
            #[inline(always)]
            fn cast_to(self) -> $t {
                self as $t
            }
        }
    };
}

#[inline(always)]
fn rem_mul<T, U>(x: T, y: T, m: T) -> T
where
    T: Copy + Cast<U>,
    U: Copy + Mul<Output = U> + Rem<Output = U> + Cast<T>,
{
    let (x, y, m): (U, U, U) = (x.cast_to(), y.cast_to(), m.cast_to());
    ((x * y) % m).cast_to()
}

#[inline]
fn rem_pow<T, U>(mut base: T, mut exp: T, m: T) -> T
where
    T: Copy + Zero + One + Eq + RemAssign + BitAnd<Output = T> + ShrAssign + Cast<U>,
    U: Copy + Mul<Output = U> + Rem<Output = U> + Cast<T>,
{
    exp %= m;
    let mut r = T::one();
    while exp != T::zero() {
        if exp & T::one() != T::zero() {
            r = rem_mul::<T, U>(r, base, m);
        }
        exp >>= T::one();
        base = rem_mul::<T, U>(base, base, m);
    }
    r
}

pub trait Miller {
    fn naive_primality(self) -> bool;
    fn miller_rabin(self, a: Self) -> bool;
    fn miller_primality(self) -> bool;
    fn is_prime(self) -> bool;
}

macro_rules! miller_rabin_impl {
    ($t:ty, $u:ty, $limit:expr,  $($x:expr),*) => {
        impl_primitive_cast!($t, $u);
        impl_primitive_cast!($u, $t);

        impl Miller for $t {
            #[inline]
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
                let mut t = rem_pow::<$t, $u>(a, p, self);
                let at_last = t == d || t == 1;

                while p!=d {
                    p <<= 1;
                    t = rem_mul::<$t, $u>(t, t, self);
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