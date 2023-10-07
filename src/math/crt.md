# Chinese Remainder Theorem

`crt(r, m)` returns `Some(x)` such that \\(x \equiv r_i \pmod {m_i}\\) for all \\(i\\). If such \\(x\\) does not exist, then it returns `None`.

## Example

```rust
# fn main() {
let r = vec![1, 2, 3];
let m = vec![3, 5, 7];
let x = crt(&r, &m);
println!("{:?}", x); // Some(52)

let r = vec![2, 5];
let m = vec![10, 25];
let x = crt(&r, &m);
println!("{:?}", x); // None
# }
# 
# fn gcd(x: i64, y: i64) -> i64 {
#     if y == 0 {
#         x
#     } else {
#         gcd(y, x % y)
#     }
# }
# 
# /// Returns `(g, s, t)` such that `g == gcd(a, b)` and `a*s + t*b == g`.
# fn egcd(mut a: i64, mut b: i64) -> (i64, i64, i64) {
#     let (mut sa, mut ta, mut sb, mut tb) = (1, 0, 0, 1);
#     while b != 0 {
#         let (q, r) = (a / b, a % b);
#         (sa, ta, sb, tb) = (sb, tb, sa - q * sb, ta - q * tb);
#         (a, b) = (b, r);
#     }
#     (a, sa, ta)
# }
# 
# /// Returns x s.t. x=a_i (mod m_i) for all i.
# /// Reference: PyRival https://github.com/cheran-senthil/PyRival/blob/master/pyrival/algebra/chinese_remainder.py
# fn crt(a: &[i64], m: &[i64]) -> Option<i64> {
#     use std::iter::zip;
#     let (mut x, mut m_prod) = (0, 1);
#     for (&ai, &mi) in zip(a, m) {
#         let (g, s, _) = egcd(m_prod, mi);
#         if (ai - x).rem_euclid(g) != 0 {
#             return None;
#         }
#         x += m_prod * ((s * ((ai - x).rem_euclid(mi))).div_euclid(g));
#         m_prod = (m_prod * mi).div_euclid(gcd(m_prod, mi));
#     }
#     Some(x.rem_euclid(m_prod))
# }
```

## Code

```rust,noplayground
fn gcd(x: i64, y: i64) -> i64 {
    if y == 0 {
        x
    } else {
        gcd(y, x % y)
    }
}

/// Returns `(g, s, t)` such that `g == gcd(a, b)` and `a*s + t*b == g`.
fn egcd(mut a: i64, mut b: i64) -> (i64, i64, i64) {
    let (mut sa, mut ta, mut sb, mut tb) = (1, 0, 0, 1);
    while b != 0 {
        let (q, r) = (a / b, a % b);
        (sa, ta, sb, tb) = (sb, tb, sa - q * sb, ta - q * tb);
        (a, b) = (b, r);
    }
    (a, sa, ta)
}

/// Returns x s.t. x=a_i (mod m_i) for all i.
/// Reference: PyRival https://github.com/cheran-senthil/PyRival/blob/master/pyrival/algebra/chinese_remainder.py
fn crt(a: &[i64], m: &[i64]) -> Option<i64> {
    use std::iter::zip;
    let (mut x, mut m_prod) = (0, 1);
    for (&ai, &mi) in zip(a, m) {
        let (g, s, _) = egcd(m_prod, mi);
        if (ai - x).rem_euclid(g) != 0 {
            return None;
        }
        x += m_prod * ((s * ((ai - x).rem_euclid(mi))).div_euclid(g));
        m_prod = (m_prod * mi).div_euclid(gcd(m_prod, mi));
    }
    Some(x.rem_euclid(m_prod))
}
```

---

Last modified on 231008.