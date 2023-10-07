# Extended Euclidean Algorithm

`egcd(a, b)` returns \\(g, s, t\\) such that \\(g = \gcd(a, b)\\) and \\(as+bt=g\\).

## Example

```rust
# fn main() {
for (a, b) in [(2, 5), (11, 17), (20, 35)] {
    let (g, s, t) = egcd(a, b);
    println!("gcd({a}, {b}) = {g}");
    println!("{a}*({s}) + {b}*({t}) = {g}");
}
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
```

## Code

```rust,noplayground
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
```

---

Last modified on 231008.