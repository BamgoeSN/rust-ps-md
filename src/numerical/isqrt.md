# Integer Square Root

```rust,noplayground
fn isqrt(s: u64) -> u64 {
    let mut x0 = s >> 1;
    if x0 != 0 {
        let mut x1 = (x0 + s / x0) >> 1;
        while x1 < x0 {
            x0 = x1;
            x1 = (x0 + s / x0) >> 1
        }
        x0
    } else {
        s
    }
}
```