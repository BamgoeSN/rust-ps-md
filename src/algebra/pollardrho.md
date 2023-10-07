# Pollard's Rho Algorithm

Pollard rho algorithm is a randomized algorithm which factorizes a number in an average time complexity of \\(O(n^{1/4})\\).

`x.factorize()` factorizes `x` and returns a vector with the factors. The order of factors in the vector is undefined.

[Miller-Rabin primality test](./millerrabin.md) should be with this snippet in the code.

## Example

```rust
use factorization::*;

# fn main() {
let mut rng = RNG::new(15163487);
let a: u64 = 484387724796727379;
let mut factors = a.factorize(&mut rng);
factors.sort_unstable();
println!("{:?}", factors); // [165551, 2925912406429]
println!("{}", factors.iter().product::<u64>()); // 484387724796727379
use millerrabin::Primality;
println!("{}", factors.iter().all(|&x| x.is_prime()));
# }
# 
# mod factorization {
#     use super::millerrabin::Primality;
#     use std::ops::*;
# 
#     pub trait PollardRho: Primality + From<u8> + PartialOrd + ShrAssign + BitAnd<Output = Self> + Clone {
#         fn rho(self, arr: &mut Vec<Self>, rng: &mut rng::RNG);
#         fn factorize(mut self, rng: &mut rng::RNG) -> Vec<Self> {
#             let mut arr: Vec<Self> = Vec::new();
#             if self <= 1.into() {
#                 return arr;
#             }
#             while self.clone() & 1.into() == 0.into() {
#                 self >>= 1.into();
#                 arr.push(2.into());
#             }
#             self.rho(&mut arr, rng);
#             arr
#         }
#     }
# 
#     macro_rules! impl_pollardrho {
#         ($t:ty, $u:ty, $reset:expr) => {
#             impl PollardRho for $t {
#                 fn rho(self, arr: &mut Vec<Self>, rng: &mut rng::RNG) {
#                     if self <= 1 {
#                         return;
#                     } else if self.is_prime() {
#                         arr.push(self);
#                         return;
#                     }
# 
#                     let mut i: u64 = 0;
#                     let mut x: $t = (rng.next_u64() % self as u64) as $t;
#                     let mut y: $t = x;
#                     let mut k: u64 = 2;
#                     let mut d: $t;
#                     let mut reset_limit: u64 = $reset;
# 
#                     loop {
#                         i += 1;
#                         x = (((x as $u * x as $u % self as $u) + (self - 1) as $u) % self as $u) as $t;
#                         d = gcd(y.abs_diff(x), self);
#                         if d == self || i >= reset_limit {
#                             // Reset
#                             reset_limit = reset_limit * 3 / 2;
#                             i = 0;
#                             x = (rng.next_u64() % self as u64) as $t;
#                             y = x;
#                         }
#                         if d != 1 {
#                             break;
#                         }
#                         if i == k {
#                             y = x;
#                             k <<= 1;
#                         }
#                     }
# 
#                     if d != self {
#                         d.rho(arr, rng);
#                         (self / d).rho(arr, rng);
#                         return;
#                     }
# 
#                     let mut i = 3;
#                     while i * i <= self {
#                         if self % i == 0 {
#                             i.rho(arr, rng);
#                             (d / i).rho(arr, rng);
#                             return;
#                         }
#                         i += 2;
#                     }
#                 }
#             }
#         };
#     }
# 
#     impl_pollardrho!(u8, u16, 100000);
#     impl_pollardrho!(u16, u32, 100000);
#     impl_pollardrho!(u32, u64, 100000);
#     impl_pollardrho!(u64, u128, 100000);
# 
#     pub fn gcd<T>(x: T, y: T) -> T
#     where
#         T: Copy + PartialEq + PartialOrd + core::ops::Rem<Output = T> + From<u8>,
#     {
#         if y == 0.into() {
#             x
#         } else {
#             let v = x % y;
#             gcd(y, v)
#         }
#     }
# 
#     pub mod rng {
#         pub struct RNG {
#             val: u64,
#         }
#         impl RNG {
#             pub fn new(seed: u64) -> Self {
#                 Self { val: seed }
#             }
#             pub fn next_u64(&mut self) -> u64 {
#                 let mut x = self.val;
#                 x ^= x << 13;
#                 x ^= x >> 7;
#                 x ^= x << 17;
#                 self.val = x;
#                 x
#             }
#         }
#     }
# 
#     pub use rng::*;
# }
# 
# mod millerrabin {
#     pub trait Primality {
#         fn is_prime(self) -> bool;
#     }
# 
#     macro_rules! impl_primality {
#         ($t:ty, $u:ty, $thres:expr, $bcnt:expr, $($basis:expr),+) => {
#             impl Primality for $t {
#                 fn is_prime(self) -> bool {
#                     if self <= 1 {
#                         return false;
#                     } else if self & 1 == 0 {
#                         return self == 2;
#                     }
# 
#                     const THRES: $t = $thres;
#                     const TEST: [$t; $bcnt] = [$($basis,)+];
# 
#                     if self <= THRES {
#                         for p in (2..).take_while(|&p| p * p <= self) {
#                             if self % p == 0 {
#                                 return false;
#                             }
#                         }
#                         return true;
#                     }
# 
#                     let pow = |base: $t, mut exp: $t| -> $t {
#                         let mut base = base as $u;
#                         let mut ret = 1 as $u;
#                         while exp != 0 {
#                             if exp & 1 != 0 {
#                                 ret = (ret * base) % self as $u;
#                             }
#                             exp >>= 1;
#                             base = (base * base) % self as $u;
#                         }
#                         ret as $t
#                     };
# 
#                     let s = (self - 1).trailing_zeros();
#                     let d = (self - 1) >> s;
# 
#                     for &a in TEST.iter().take_while(|&&a| a < self - 1) {
#                         let mut x = pow(a, d);
#                         for _ in 0..s {
#                             let y = ((x as $u).pow(2) % self as $u) as $t;
#                             if y == 1 && x != 1 && x != self - 1 {
#                                 return false;
#                             }
#                             x = y;
#                         }
#                         if x != 1 {
#                             return false;
#                         }
#                     }
# 
#                     true
#                 }
#             }
#         };
#     }
# 
#     impl_primality!(u8, u16, 255, 1, 2);
#     impl_primality!(u16, u32, 2000, 2, 2, 3);
#     impl_primality!(u32, u64, 7000, 3, 2, 7, 61);
#     impl_primality!(u64, u128, 300000, 7, 2, 325, 9375, 28178, 450775, 9780504, 1795265022);
# }
```

## Code

```rust,noplayground
mod factorization {
    use super::millerrabin::Primality;
    use std::ops::*;

    pub trait PollardRho: Primality + From<u8> + PartialOrd + ShrAssign + BitAnd<Output = Self> + Clone {
        fn rho(self, arr: &mut Vec<Self>, rng: &mut rng::RNG);
        fn factorize(mut self, rng: &mut rng::RNG) -> Vec<Self> {
            let mut arr: Vec<Self> = Vec::new();
            if self <= 1.into() {
                return arr;
            }
            while self.clone() & 1.into() == 0.into() {
                self >>= 1.into();
                arr.push(2.into());
            }
            self.rho(&mut arr, rng);
            arr
        }
    }

    macro_rules! impl_pollardrho {
        ($t:ty, $u:ty, $reset:expr) => {
            impl PollardRho for $t {
                fn rho(self, arr: &mut Vec<Self>, rng: &mut rng::RNG) {
                    if self <= 1 {
                        return;
                    } else if self.is_prime() {
                        arr.push(self);
                        return;
                    }

                    let mut i: u64 = 0;
                    let mut x: $t = (rng.next_u64() % self as u64) as $t;
                    let mut y: $t = x;
                    let mut k: u64 = 2;
                    let mut d: $t;
                    let mut reset_limit: u64 = $reset;

                    loop {
                        i += 1;
                        x = (((x as $u * x as $u % self as $u) + (self - 1) as $u) % self as $u) as $t;
                        d = gcd(y.abs_diff(x), self);
                        if d == self || i >= reset_limit {
                            // Reset
                            reset_limit = reset_limit * 3 / 2;
                            i = 0;
                            x = (rng.next_u64() % self as u64) as $t;
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

                    if d != self {
                        d.rho(arr, rng);
                        (self / d).rho(arr, rng);
                        return;
                    }

                    let mut i = 3;
                    while i * i <= self {
                        if self % i == 0 {
                            i.rho(arr, rng);
                            (d / i).rho(arr, rng);
                            return;
                        }
                        i += 2;
                    }
                }
            }
        };
    }

    impl_pollardrho!(u8, u16, 100000);
    impl_pollardrho!(u16, u32, 100000);
    impl_pollardrho!(u32, u64, 100000);
    impl_pollardrho!(u64, u128, 100000);

    pub fn gcd<T>(x: T, y: T) -> T
    where
        T: Copy + PartialEq + PartialOrd + core::ops::Rem<Output = T> + From<u8>,
    {
        if y == 0.into() {
            x
        } else {
            let v = x % y;
            gcd(y, v)
        }
    }

    pub mod rng {
        pub struct RNG {
            val: u64,
        }
        impl RNG {
            pub fn new(seed: u64) -> Self {
                Self { val: seed }
            }
            pub fn next_u64(&mut self) -> u64 {
                let mut x = self.val;
                x ^= x << 13;
                x ^= x >> 7;
                x ^= x << 17;
                self.val = x;
                x
            }
        }
    }

    pub use self::rng::*;
}
```

---

Last updated on 231008.