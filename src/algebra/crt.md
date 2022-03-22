# Chinese Remainder Theorem
Reference: PyRival <https://github.com/cheran-senthil/PyRival/blob/master/pyrival/algebra/chinese_remainder.py>

Returns \\(x\\) such that \\(x = r_i \mod m_i\\) for all \\(i\\). \

Required snippets: [GCD](gcd.md#gcd), [Extended Euclidean Algorithm](extgcd.md#extended-euclid-algorithm)
```rust,noplayground
// Chinese remainder theorem
//
// Reference: PyRival <https://github.com/cheran-senthil/PyRival/blob/master/pyrival/algebra/chinese_remainder.py>

#[inline(always)]
fn safe_mod(mut x: i64, m: i64) -> i64 {
    x %= m;
    if x < 0 {
        x + m
    } else {
        x
    }
}

/// Returns the greatest common divisor of a and b
fn gcd(mut a: i64, mut b: i64) -> i64 {
    while b != 0 {
        let t = b;
        b = a % b;
        a = t;
    }
    a
}

/// Returns gcd(a, b), s, r s.t. a*s + b*r = gcd(a, b)
fn ext_gcd(a: i64, b: i64) -> (i64, i64, i64) {
    let (mut s, mut old_s) = (0, 1);
    let (mut r, mut old_r) = (b, a);
    while r != 0 {
        let q = old_r / r;

        let new_r = old_r - q * r;
        old_r = r;
        r = new_r;

        let new_s = old_s - q * s;
        old_s = s;
        s = new_s;
    }

    (
        old_r,
        old_s,
        if b != 0 { (old_r - old_s * a) / b } else { 0 },
    )
}

/// Returns $x$ s.t. $x=r_i (mod m_i)$ for all $i$
pub fn crt(r: &[i64], m: &[i64]) -> Option<i64> {
    let (mut x, mut m_prod) = (0, 1);
    for (bi, mi) in r.iter().zip(m.iter()) {
        let (g, s, _) = ext_gcd(m_prod, *mi);
        if safe_mod((bi - x) % mi, g) != 0 {
            return None;
        }
        x += m_prod * (s * safe_mod(bi - x, *mi) / g);
        m_prod = (m_prod * mi) / gcd(m_prod, *mi);
    }
    Some(safe_mod(x, m_prod))
}
```