# Line Intersection

## Code

```rust,noplayground
type Frac = [i128; 2];
/// Calculates the intersection of line segments `p` and `q`.
/// Returns `None` for no intersection.
/// Returns `Some(Ok([x, y]))` for a point intersection, where `[x, y]` is a fraction (`[numerator, denominator]`).
/// Returns `Some(Err([x, y]))` for an overlapping line segment, where `[x, y]` represents endpoints of the overlap.
///
/// Note: Potential overflow for `i64` or `i128`; intended for use with `i32`.
fn intersect(p: L, q: L) -> Option<Result<[Frac; 2], L>> {
	use std::cmp::Ordering::*;
	let u = cross(sub(p[1], p[0]), sub(q[1], q[0]));
	let sn = cross(sub(q[0], p[0]), sub(q[1], q[0]));
	let tn = cross(sub(q[0], p[0]), sub(p[1], p[0]));
	if u != 0 {
		let int = if u >= 0 { 0..=u } else { u..=0 };
		if int.contains(&sn) && int.contains(&tn) {
			let [s, r] = [sn, u - sn].map(|x| x as i128);
			let [g, h] = p.map(|f| f.map(|x| x as i128));
			let [x, y] = [0, 1].map(|i| [r * g[i] + s * h[i], u as i128]);
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

The code below only checks if two lines meet, without calculating where they do.

```rust,noplayground
/// Checks if line segments `p` and `q` intersect.
/// Returns `true` if they intersect at any point, `false` otherwise.
fn meets(p: L, q: L) -> bool {
	let u = cross(sub(p[1], p[0]), sub(q[1], q[0]));
	let sn = cross(sub(q[0], p[0]), sub(q[1], q[0]));
	let tn = cross(sub(q[0], p[0]), sub(p[1], p[0]));
	if u != 0 {
		let int = if u >= 0 { 0..=u } else { u..=0 };
		int.contains(&sn) && int.contains(&tn)
	} else {
		if sn != 0 || tn != 0 {
			return false;
		}
		let (a0, a1) = (p[0].min(p[1]), p[0].max(p[1]));
		let (b0, b1) = (q[0].min(q[1]), q[0].max(q[1]));
		let (l, r) = (a0.max(b0), a1.min(b1));
		l <= r
	}
}
```

---

Last modified on 240119.