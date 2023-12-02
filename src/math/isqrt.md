# Integer Square Root

`isqrt(s)` returns \\( \left\lfloor \sqrt{s} \right\rfloor \\). It runs much faster than the typical binary search method, but slower than casting the result from `std::f64::sqrt`. If the value can be perfectly represented with `f64` and the memory limit isn't too short, it's better to use the `f64` square root function from `std`.

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
fn isqrt<T>(s: T) -> T
where T: Copy + Shr<Output = T> + Add<Output = T> + Div<Output = T> + PartialOrd + From<u8> {
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

---

Last modified on 231203.