# GCD
Returns the greatest common divisor of a and b.
```rust,noplayground
pub fn gcd(mut a: i64, mut b: i64) -> i64 {
    while b != 0 {
        let t = b;
        b = a % b;
        a = t;
    }
    a
}
```

# LCM
Returns the least common multiplier of a and b. \
Required snippets: [GCD](#gcd)
```rust,noplayground
pub fn lcm(a: i64, b: i64) -> i64 {
    a / gcd(a, b) * b
}
```

