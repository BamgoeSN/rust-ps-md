# Line Intersection

## Code

```rust,noplayground
type I = i32;
type P = [I; 2];
type L = [P; 2];
type Frac = [i128; 2];

fn sub(a: P, b: P) -> P { [a[0] - b[0], a[1] - b[1]] }
fn cross(a: P, b: P) -> i128 { a[0] as i128 * b[1] as i128 - a[1] as i128 * b[0] as i128 }

/// Returns `None` if `p` and `q` don't meet.
///
/// Returns `Some(Ok([x, y]))` if `p` and `q` meet in a single point `(x, y)`.
/// Here, `x` and `y` have a type of `[i128; 2]` which indicates a fraction, where its first element is a numerator and the second element is a denominator.
///
/// Returns `Some(Err([x, y]))` if `p` and `q` overlaps at a line segment.
/// Here, `x` and `y` represent both ends of the segment.
fn intersect(p: L, q: L) -> Option<Result<[Frac; 2], L>> {
	use std::cmp::Ordering::*;
	let u = cross(sub(p[1], p[0]), sub(q[1], q[0]));
	let sn = cross(sub(q[0], p[0]), sub(q[1], q[0]));
	let tn = cross(sub(q[0], p[0]), sub(p[1], p[0]));
	if u != 0 {
		let int = if u >= 0 { 0..=u } else { u..=0 };
		if int.contains(&sn) && int.contains(&tn) {
			let (s, r) = (sn, u - sn);
			let [g, h] = p.map(|f| f.map(|x| x as i128));
			let [x, y] = [0, 1].map(|i| [r * g[i] + s * h[i], u]);
			Some(Ok([x, y]))
		} else {
			None
		}
	} else {
		if sn != 0 || tn != 0 {
			return None;
		}
		let (a0, a1) = (p[0].min(p[1]), p[0].max(p[1]));
		let (b0, b1) = (q[0].min(q[1]), q[0].max(q[1]));
		let (l, r) = (a0.max(b0), a1.min(b1));
		match l.cmp(&r) {
			Less => Some(Err([l, r])),
			Equal => Some(Ok(l.map(|x| [x.into(), 1]))),
			Greater => None,
		}
	}
}
```

---

Last modified on 231203.