# GCD, LCM

`gcd(x, y)` returns the greatest common divisor (GCD) of `x` and `y`. \
`lcm(x, y)` returns the least common multiple (LCM) of `x` and `y`.

`gcd` is implemented using Euclidean algorithm, whose time complexity is \\(O( \log \_{\phi} x )\\) where \\(\phi\\) is a golden ratio.

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

## Example

```rust
#fn main() {
let (x, y) = (10, 25);

let g = gcd(x, y);
println!("{}", g); // 5

let l = lcm(x, y);
println!("{}", l); // 50
#}
#
#fn gcd(x: u64, y: u64) -> u64 {
#    if y == 0 {
#        x
#    } else {
#        gcd(y, x % y)
#    }
#}
#
#fn lcm(x: u64, y: u64) -> u64 {
#    x / gcd(x, y) * y
#}
```
