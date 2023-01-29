# Deterministic Miller-Rabin Primality Test

Deterministic Miller-Rabin primality test determines whether a certain unsigned integer is a prime in a time complexity of \\(O(\log{n})\\). This test only works for integers under \\(2^{64}\\).

`x.is_prime()` chooses a roughly faster algorithm among naive primality test and Miller-Rabin test, and returns `true` if `x` is a prime, `false` if not.

## Example

```rust
use miller_rabin::MillerRabin;

# 
# fn main() {
println!("{}", 3284729387909u64.is_prime()); // true
println!("{}", 3284729387911u64.is_prime()); // false 53×61976026187
# }
# 
# mod miller_rabin {
#     pub trait MillerRabin: From<u8> + PartialOrd {
#         const MR_THRES: Self;
#         fn naive_primality(self) -> bool;
#         fn miller_rabin_test(self, a: Self) -> bool;
#         fn miller_primality(self) -> bool;
#         fn is_prime(self) -> bool {
#             if self <= 1.into() {
#                 false
#             } else if self <= Self::MR_THRES {
#                 self.naive_primality()
#             } else {
#                 self.miller_primality()
#             }
#         }
#     }
# 
#     macro_rules! impl_millerrabin {
#         ($t:ty, $u:ty, $thres:expr, $($x:expr),*) => {
#             impl MillerRabin for $t {
#                 const MR_THRES: Self = $thres;
# 
#                 #[inline(always)]
#                 fn naive_primality(self) -> bool {
#                     (2..).take_while(|&i| i * i <= self).all(|i| self % i != 0)
#                 }
# 
#                 #[inline(always)]
#                 fn miller_rabin_test(self, a: Self) -> bool {
#                     let d = self - 1;
#                     let mut p = d >> (d.trailing_zeros());
# 
#                     let mut t = {
#                         let mut base = a as $u;
#                         let mut exp = p as $u;
#                         let rem = self as $u;
#                         let mut ret: $u = 1;
#                         while exp != 0 {
#                             if exp & 1 != 0 {
#                                 ret = ret * base % rem;
#                             }
#                             base = base * base % rem;
#                             exp >>= 1;
#                         }
#                         ret as $t
#                     };
# 
#                     let at_last = t == d || t == 1;
# 
#                     while p != d {
#                         p <<= 1;
#                         t = ((t as $u * t as $u) % self as $u) as $t;
#                         if t == self - 1 {
#                             return true;
#                         }
#                     }
#                     at_last
#                 }
# 
#                 fn miller_primality(self) -> bool {
#                     $(
#                         if !self.miller_rabin_test($x) { return false; }
#                     )*
#                     true
#                 }
#             }
#         };
#     }
# 
#     impl_millerrabin!(u8, u16, 254, 2);
#     impl_millerrabin!(u16, u32, 2000, 2, 3);
#     impl_millerrabin!(u32, u64, 7000, 2, 7, 61);
#     impl_millerrabin!(u64, u128, 300000, 2, 325, 9375, 28178, 450775, 9780504, 1795265022);
# }
```

## Code

```rust,noplayground
mod miller_rabin {
    pub trait MillerRabin: From<u8> + PartialOrd {
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
                    (2..).take_while(|&i| i * i <= self).all(|i| self % i != 0)
                }

                #[inline(always)]
                fn miller_rabin_test(self, a: Self) -> bool {
                    let d = self - 1;
                    let mut p = d >> (d.trailing_zeros());

                    let mut t = {
                        let mut base = a as $u;
                        let mut exp = p as $u;
                        let rem = self as $u;
                        let mut ret: $u = 1;
                        while exp != 0 {
                            if exp & 1 != 0 {
                                ret = ret * base % rem;
                            }
                            base = base * base % rem;
                            exp >>= 1;
                        }
                        ret as $t
                    };

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
                        if !self.miller_rabin_test($x) { return false; }
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
}
```
