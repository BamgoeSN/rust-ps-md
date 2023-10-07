# Deterministic Miller-Rabin Primality Test

Deterministic Miller-Rabin primality test determines whether a certain unsigned integer is a prime in a time complexity of \\(O(\log{n})\\). This test only works for integers under \\(2^{64}\\).

`x.is_prime()` chooses a roughly faster algorithm among naive primality test and Miller-Rabin test, and returns `true` if `x` is a prime, `false` if not.

## Example

```rust
use millerrabin::*;

# fn main() {
println!("{}", 407521u64.is_prime()); // true
println!("{}", 3284729387909u64.is_prime()); // true
println!("{}", 3284729387911u64.is_prime()); // false 53×61976026187
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
mod millerrabin {
    pub trait Primality {
        fn is_prime(self) -> bool;
    }

    macro_rules! impl_primality {
        ($t:ty, $u:ty, $thres:expr, $bcnt:expr, $($basis:expr),+) => {
            impl Primality for $t {
                fn is_prime(self) -> bool {
                    if self <= 1 {
                        return false;
                    } else if self & 1 == 0 {
                        return self == 2;
                    }

                    const THRES: $t = $thres;
                    const TEST: [$t; $bcnt] = [$($basis,)+];

                    if self <= THRES {
                        for p in (2..).take_while(|&p| p * p <= self) {
                            if self % p == 0 {
                                return false;
                            }
                        }
                        return true;
                    }

                    let pow = |base: $t, mut exp: $t| -> $t {
                        let mut base = base as $u;
                        let mut ret = 1 as $u;
                        while exp != 0 {
                            if exp & 1 != 0 {
                                ret = (ret * base) % self as $u;
                            }
                            exp >>= 1;
                            base = (base * base) % self as $u;
                        }
                        ret as $t
                    };

                    let s = (self - 1).trailing_zeros();
                    let d = (self - 1) >> s;

                    for &a in TEST.iter().take_while(|&&a| a < self - 1) {
                        let mut x = pow(a, d);
                        for _ in 0..s {
                            let y = ((x as $u).pow(2) % self as $u) as $t;
                            if y == 1 && x != 1 && x != self - 1 {
                                return false;
                            }
                            x = y;
                        }
                        if x != 1 {
                            return false;
                        }
                    }

                    true
                }
            }
        };
    }

    impl_primality!(u8, u16, 255, 1, 2);
    impl_primality!(u16, u32, 2000, 2, 2, 3);
    impl_primality!(u32, u64, 7000, 3, 2, 7, 61);
    impl_primality!(u64, u128, 300000, 7, 2, 325, 9375, 28178, 450775, 9780504, 1795265022);
}
```

---

Last updated on 231008.