# 2D Geometry Base

## Code
```rust,noplayground
type I = i64;
type P = [I; 2];
type L = [P; 2];

fn scale(s: I, a: P) -> P { a.map(|x| x * s) }
fn add(a: P, b: P) -> P { [a[0] + b[0], a[1] + b[1]] }
fn sub(a: P, b: P) -> P { [a[0] - b[0], a[1] - b[1]] }
fn dot(a: P, b: P) -> I { a[0] * b[0] + a[1] * b[1] }
fn cross(a: P, b: P) -> I { a[0] * b[1] - a[1] * b[0] }
fn ccw(a: P, b: P, c: P) -> I { cross(sub(b, a), sub(c, b)) }
```

---

Last modified on 231225.