# Pollard's Rho Algorithm

Pollard rho algorithm is a randomized algorithm which factorizes a number in an average time complexity of \\(O(n^{1/4})\\).

`x.factorize()` factorizes `x` and returns a vector with the factors. The order of factors in the vector is undefined.

## Example

```rust
# fn main() {
let mut rng = rng::RNG::new(12345);
let a: u32 = 1237172;
let mut factors = a.factorize(&mut rng);
factors.sort_unstable();
println!("{:?}", factors); // [2, 2, 309293]
println!("{}", factors.iter().product::<u32>()); // 1237172
# }
#
# trait PollardRho: MillerRabin + std::ops::ShrAssign + std::ops::BitAnd<Output = Self> + Clone {
#     fn rho(self, arr: &mut Vec<Self>, rng: &mut rng::RNG);
#     fn factorize(mut self, rng: &mut rng::RNG) -> Vec<Self> {
#         let mut arr: Vec<Self> = Vec::new();
#         if self <= 1.into() {
#             return arr;
#         }
#         while self.clone() & 1.into() == 0.into() {
#             self >>= 1.into();
#             arr.push(2.into());
#         }
#         self.rho(&mut arr, rng);
#         arr
#     }
# }
#
# macro_rules! impl_pollardrho {
#     ($t:ty, $u:ty, $reset:expr) => {
#         impl PollardRho for $t {
#             fn rho(self, arr: &mut Vec<Self>, rng: &mut rng::RNG) {
#                 if self <= 1 {
#                     return;
#                 } else if self.is_prime() {
#                     arr.push(self);
#                     return;
#                 }
#
#                 let mut i: u64 = 0;
#                 let mut x: $t = (rng.next_u64() % self as u64) as $t;
#                 let mut y: $t = x;
#                 let mut k: u64 = 2;
#                 let mut d: $t;
#                 let mut reset_limit: u64 = $reset;
#
#                 loop {
#                     i += 1;
#                     x = (((x as $u * x as $u % self as $u) + (self - 1) as $u) % self as $u) as $t;
#                     d = gcd(y.abs_diff(x), self);
#                     if d == self || i >= $reset {
#                         // Reset
#                         reset_limit = reset_limit * 3 / 2;
#                         i = 0;
#                         x = (rng.next_u64() % self as u64) as $t;
#                         y = x;
#                     }
#                     if d != 1 {
#                         break;
#                     }
#                     if i == k {
#                         y = x;
#                         k <<= 1;
#                     }
#                 }
#
#                 if d != self {
#                     d.rho(arr, rng);
#                     (self / d).rho(arr, rng);
#                     return;
#                 }
#
#                 let mut i = 3;
#                 while i * i <= self {
#                     if self % i == 0 {
#                         i.rho(arr, rng);
#                         (d / i).rho(arr, rng);
#                         return;
#                     }
#                     i += 2;
#                 }
#             }
#         }
#     };
# }
#
# impl_pollardrho!(u8, u16, 100000);
# impl_pollardrho!(u16, u32, 100000);
# impl_pollardrho!(u32, u64, 100000);
# impl_pollardrho!(u64, u128, 100000);
#
# trait MillerRabin: From<u8> + PartialOrd {
#     const MR_THRES: Self;
#     fn naive_primality(self) -> bool;
#     fn miller_rabin_test(self, a: Self) -> bool;
#     fn miller_primality(self) -> bool;
#     fn is_prime(self) -> bool {
#         if self <= 1.into() {
#             false
#         } else if self <= Self::MR_THRES {
#             self.naive_primality()
#         } else {
#             self.miller_primality()
#         }
#     }
# }
#
# macro_rules! impl_millerrabin {
#     ($t:ty, $u:ty, $thres:expr, $($x:expr),*) => {
#         impl MillerRabin for $t {
#             const MR_THRES: Self = $thres;
#
#             #[inline(always)]
#             fn naive_primality(self) -> bool {
#                 for i in (2..).take_while(|&i| i * i <= self) {
#                     if self % i == 0 {
#                         return false;
#                     }
#                 }
#                 true
#             }
#
#             #[inline(always)]
#             fn miller_rabin_test(self, a: Self) -> bool {
#                 macro_rules! rem_pow {
#                     ($base:expr, $exp:expr, $rem:expr, $tx:ty, $ux:ty) => {{
#                         let mut base = $base as $ux;
#                         let mut exp = $exp as $ux;
#                         let rem = $rem as $ux;
#                         let mut ret: $ux = 1;
#                         while exp != 0 {
#                             if exp & 1 != 0 {
#                                 ret = ret * base % rem;
#                             }
#                             base = base * base % rem;
#                             exp >>= 1;
#                         }
#                         ret as $tx
#                     }};
#                 }
#
#                 let d = self - 1;
#                 let mut p = d >> (d.trailing_zeros());
#                 let mut t = rem_pow!(a, p, self, $t, $u);
#                 let at_last = t == d || t == 1;
#
#                 while p != d {
#                     p <<= 1;
#                     t = ((t as $u * t as $u) % self as $u) as $t;
#                     if t == self - 1 {
#                         return true;
#                     }
#                 }
#                 at_last
#             }
#
#             fn miller_primality(self) -> bool {
#                 $(
#                     if !self.miller_rabin_test($x) {
#                         return false;
#                     }
#                 )*
#                 true
#             }
#         }
#     };
# }
#
# impl_millerrabin!(u8, u16, 254, 2);
# impl_millerrabin!(u16, u32, 2000, 2, 3);
# impl_millerrabin!(u32, u64, 7000, 2, 7, 61);
# impl_millerrabin!(u64, u128, 300000, 2, 325, 9375, 28178, 450775, 9780504, 1795265022);
#
# pub fn gcd<T>(x: T, y: T) -> T
# where
#     T: Copy + PartialEq + PartialOrd + std::ops::Rem<Output = T> + From<u8>,
# {
#     if y == 0.into() {
#         x
#     } else {
#         let v = x % y;
#         gcd(y, v)
#     }
# }
#
# mod rng {
#     pub struct RNG {
#         val: u64,
#     }
#     impl RNG {
#         pub fn new(seed: u64) -> Self {
#             Self { val: seed }
#         }
#         pub fn next_u64(&mut self) -> u64 {
#             let mut x = self.val;
#             x ^= x << 13;
#             x ^= x >> 7;
#             x ^= x << 17;
#             self.val = x;
#             x
#         }
#     }
# }
```

## Code

```rust,noplayground
trait PollardRho: MillerRabin + std::ops::ShrAssign + std::ops::BitAnd<Output = Self> + Clone {
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
                    if d == self || i >= $reset {
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

trait MillerRabin: From<u8> + PartialOrd {
    const MR_THRES: Self;
    fn naive_primality(self) -> bool;
    fn miller_rabin_test(self, a: Self) -> bool;
    fn miller_primality(self) -> bool;
    fn is_prime(self) -> bool {
        if self <= 1.into() {
            false
        } else if self <= Self::MR_THRES {
            self.naive_primality()
        } else {
            self.miller_primality()
        }
    }
}

macro_rules! impl_millerrabin {
    ($t:ty, $u:ty, $thres:expr, $($x:expr),*) => {
        impl MillerRabin for $t {
            const MR_THRES: Self = $thres;

            #[inline(always)]
            fn naive_primality(self) -> bool {
                for i in (2..).take_while(|&i| i * i <= self) {
                    if self % i == 0 {
                        return false;
                    }
                }
                true
            }

            #[inline(always)]
            fn miller_rabin_test(self, a: Self) -> bool {
                macro_rules! rem_pow {
                    ($base:expr, $exp:expr, $rem:expr, $tx:ty, $ux:ty) => {{
                        let mut base = $base as $ux;
                        let mut exp = $exp as $ux;
                        let rem = $rem as $ux;
                        let mut ret: $ux = 1;
                        while exp != 0 {
                            if exp & 1 != 0 {
                                ret = ret * base % rem;
                            }
                            base = base * base % rem;
                            exp >>= 1;
                        }
                        ret as $tx
                    }};
                }

                let d = self - 1;
                let mut p = d >> (d.trailing_zeros());
                let mut t = rem_pow!(a, p, self, $t, $u);
                let at_last = t == d || t == 1;

                while p != d {
                    p <<= 1;
                    t = ((t as $u * t as $u) % self as $u) as $t;
                    if t == self - 1 {
                        return true;
                    }
                }
                at_last
            }

            fn miller_primality(self) -> bool {
                $(
                    if !self.miller_rabin_test($x) {
                        return false;
                    }
                )*
                true
            }
        }
    };
}

impl_millerrabin!(u8, u16, 254, 2);
impl_millerrabin!(u16, u32, 2000, 2, 3);
impl_millerrabin!(u32, u64, 7000, 2, 7, 61);
impl_millerrabin!(u64, u128, 300000, 2, 325, 9375, 28178, 450775, 9780504, 1795265022);

pub fn gcd<T>(x: T, y: T) -> T
where
    T: Copy + PartialEq + PartialOrd + std::ops::Rem<Output = T> + From<u8>,
{
    if y == 0.into() {
        x
    } else {
        let v = x % y;
        gcd(y, v)
    }
}

mod rng {
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
```
