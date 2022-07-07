# Chinese Remainder Theorem

`crt(r, m)` returns `Some(x)` such that \\(x \equiv r_i \pmod {m_i}\\) for all \\(i\\). If such \\(x\\) does not exist, then it returns `None`.

## Example

```rust
# fn main() {
let r: Vec<i64> = vec![1, 2, 3];
let m: Vec<i64> = vec![3, 5, 7];
let x = crt(&r, &m);
println!("{:?}", x); // Some(52)

let r: Vec<i64> = vec![2, 5];
let m: Vec<i64> = vec![10, 25];
let x = crt(&r, &m);
println!("{:?}", x); // None
# }
#
# // Chinese remainder theorem
# // Reference: PyRival <https://github.com/cheran-senthil/PyRival/blob/master/pyrival/algebra/chinese_remainder.py>
#
# fn gcd(x: i64, y: i64) -> i64 {
#     if y == 0 {
#         x
#     } else {
#         gcd(y, x % y)
#     }
# }
#
# /// Returns gcd(a, b), s, r s.t. a*s + b*r = gcd(a, b)
# #[inline(always)]
# fn ext_gcd(a: i64, b: i64) -> (i64, i64, i64) {
#     let (mut s, mut old_s) = (0, 1);
#     let (mut r, mut old_r) = (b, a);
#     while r != 0 {
#         let q = old_r / r;
#         let (new_r, new_s) = (old_r - q * r, old_s - q * s);
#         old_r = r; // Not using destructuring to support low version
#         r = new_r; // AtCoder is using 1.42.0
#         old_s = s;
#         s = new_s;
#     }
#
#     (
#         old_r,
#         old_s,
#         if b != 0 { (old_r - old_s * a) / b } else { 0 },
#     )
# }
#
# /// Returns $x$ s.t. $x=r_i (mod m_i)$ for all $i$
# fn crt(r: &[i64], m: &[i64]) -> Option<i64> {
#     let (mut x, mut m_prod) = (0, 1);
#     for (bi, mi) in r.iter().zip(m.iter()) {
#         let (g, s, _) = ext_gcd(m_prod, *mi);
#         if ((bi - x) % mi).rem_euclid(g) != 0 {
#             return None;
#         }
#         x += m_prod * ((s * ((bi - x).rem_euclid(*mi))).div_euclid(g));
#         m_prod = (m_prod * mi).div_euclid(gcd(m_prod, *mi));
#     }
#     Some(x.rem_euclid(m_prod))
# }
```

## Code

```rust,noplayground
// Chinese remainder theorem
// Reference: PyRival https://github.com/cheran-senthil/PyRival/blob/master/pyrival/algebra/chinese_remainder.py

fn gcd(x: i64, y: i64) -> i64 {
    if y == 0 {
        x
    } else {
        gcd(y, x % y)
    }
}

/// Returns gcd(a, b), s, t s.t. a*s + b*t = gcd(a, b)
#[inline(always)]
fn ext_gcd(a: i64, b: i64) -> (i64, i64, i64) {
    let (mut s, mut old_s) = (0, 1);
    let (mut r, mut old_r) = (b, a);
    while r != 0 {
        let q = old_r / r;
        let (new_r, new_s) = (old_r - q * r, old_s - q * s);
        old_r = r; // Not using destructuring to support low version
        r = new_r; // AtCoder is using 1.42.0
        old_s = s;
        s = new_s;
    }

    (
        old_r,
        old_s,
        if b != 0 { (old_r - old_s * a) / b } else { 0 },
    )
}

/// Returns x s.t. x=r_i (mod m_i) for all i
fn crt(r: &[i64], m: &[i64]) -> Option<i64> {
    let (mut x, mut m_prod) = (0, 1);
    for (bi, mi) in r.iter().zip(m.iter()) {
        let (g, s, _) = ext_gcd(m_prod, *mi);
        if ((bi - x) % mi).rem_euclid(g) != 0 {
            return None;
        }
        x += m_prod * ((s * ((bi - x).rem_euclid(*mi))).div_euclid(g));
        m_prod = (m_prod * mi).div_euclid(gcd(m_prod, *mi));
    }
    Some(x.rem_euclid(m_prod))
}
```
