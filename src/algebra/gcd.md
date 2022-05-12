# GCD, LCM

```rust,noplayground
fn gcd(x: u64, y: u64) -> u64 {
    if y == 0 {
        x
    } else {
        gcd(y, x % y)
    }
}

fn lcm(x: u64, y: u64) -> u64 {
    x / gcd(x, y) * y
}
```
