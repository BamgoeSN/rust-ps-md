# GCD, LCM

`gcd(x, y)` returns the greatest common divisor (GCD) of `x` and `y`. \
`lcm(x, y)` returns the least common multiple (LCM) of `x` and `y`.

`gcd` is implemented using Euclidean algorithm, whose time complexity is \\(O( \log \_{\phi} x )\\) where \\(\phi\\) is a golden ratio.

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

## Code

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

## Generic Version

The function below works for any primitive unsigned integer types.

```rust,noplayground
fn gcd<T>(x: T, y: T) -> T
where T: Copy + PartialEq + PartialOrd + Rem<Output = T> + From<u8> {
	if y == 0.into() {
		x
	} else {
		let v = x % y;
		gcd(y, v)
	}
}

fn lcm<T>(x: T, y: T) -> T
where T: Copy + PartialEq + PartialOrd + Rem<Output = T> + Div<Output = T> + Mul<Output = T> + From<u8> {
	x / gcd(x, y) * y
}
```

---

Last modified on 231203.