# Extended Euclid Algorithm
Returns \\(r, s, t\\) such that \\(r = \gcd(a, b)\\) and \\(as+bt=r\\).
```rust,noplayground
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
```


