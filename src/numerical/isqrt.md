# Integer Square Root
Returns \\( \left\lfloor \sqrt{n} \right\rfloor \\).

`isqrt(s)` returns \\( \left\lfloor \sqrt{s} \right\rfloor \\). It runs much faster than the typical binary search method.

## Example

```rust
# fn main() {
let x: u64 = 10002;
let sq = isqrt(x);
println!("{}", sq); // 100
# }
#
# fn isqrt(s: u64) -> u64 {
#     let mut x0 = s >> 1;
#     if x0 != 0 {
#         let mut x1 = (x0 + s / x0) >> 1;
#         while x1 < x0 {
#             x0 = x1;
#             x1 = (x0 + s / x0) >> 1
#         }
#         x0
#     } else {
#         s
#     }
# }
```

## Code

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

## Generic Version

The function below works for any primitive unsigned integer types.

```rust,noplayground
fn isqrt<T>(s: T) -> T
where
    T: Copy
        + std::ops::Shr<Output = T>
        + std::ops::Add<Output = T>
        + std::ops::Div<Output = T>
        + PartialOrd
        + From<u8>,
{
    let mut x0 = s >> 1.into();
    if x0 != 0.into() {
        let mut x1 = (x0 + s / x0) >> 1.into();
        while x1 < x0 {
            x0 = x1;
            x1 = (x0 + s / x0) >> 1.into();
        }
        x0
    } else {
        s
    }
}
```
