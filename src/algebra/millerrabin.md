# Miller-Rabin Primality Test

Deterministic Miller-Rabin primality test determines whether a certain unsigned integer is a prime or not within \\(O(\log{n})\\) time. This test only works for integers under \\(2^{64}\\).

```rust,noplayground
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
```

## Example

```rust
# fn main() {
println!("{}", 3284729387909u64.is_prime());
println!("{}", 3284729387911u64.is_prime()); // 53×61976026187
# }
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
```
