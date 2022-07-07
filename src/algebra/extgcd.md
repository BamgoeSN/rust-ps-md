# Extended Euclidean Algorithm

`ext_gcd(a, b)` returns \\(g, s, t\\) such that \\(g = \gcd(a, b)\\) and \\(as+bt=g\\).

```rust,noplayground
// Extended Euclidean Algorithm
// Reference: PyRival https://github.com/cheran-senthil/PyRival/blob/master/pyrival/algebra/gcd.py

fn ext_gcd(a: i64, b: i64) -> (i64, i64, i64) {
    let (mut s, mut old_s) = (0, 1);
    let (mut g, mut old_g) = (b, a);
    while g != 0 {
        let q = old_g / g;
        let (new_r, new_s) = (old_g - q * g, old_s - q * s);
        old_g = g; // Not using destructuring to support low version
        g = new_r; // AtCoder is using 1.42.0
        old_s = s;
        s = new_s;
    }

    (
        old_g,
        old_s,
        if b != 0 { (old_g - old_s * a) / b } else { 0 },
    )
}
```

## Example

```rust
# fn main() {
let (a, b) = (4, 11);
let (g, s, t) = ext_gcd(a, b);
println!("{} {} {}", g, s, t); // 1 3 -1
println!("{} == {}", g, a * s + b * t);
# }
#
# // Extended Euclidean Algorithm
# // Reference: PyRival https://github.com/cheran-senthil/PyRival/blob/master/pyrival/algebra/gcd.py
#
# fn ext_gcd(a: i64, b: i64) -> (i64, i64, i64) {
#     let (mut s, mut old_s) = (0, 1);
#     let (mut g, mut old_g) = (b, a);
#     while g != 0 {
#         let q = old_g / g;
#         let (new_r, new_s) = (old_g - q * g, old_s - q * s);
#         old_g = g; // Not using destructuring to support low version
#         g = new_r; // AtCoder is using 1.42.0
#         old_s = s;
#         s = new_s;
#     }
#
#     (
#         old_g,
#         old_s,
#         if b != 0 { (old_g - old_s * a) / b } else { 0 },
#     )
# }
```
