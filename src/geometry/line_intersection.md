# Line Intersection

## Code

```rust,noplayground
type Point = [i64; 2];
type Line = [Point; 2];
type Frac = [i128; 2]; // [numerator, denominator]

fn sub(a: Point, b: Point) -> Point { [a[0] - b[0], a[1] - b[1]] }
fn cross(a: Point, b: Point) -> i128 { a[0] as i128 * b[1] as i128 - a[1] as i128 * b[0] as i128 }
fn ccw(a: Point, b: Point, c: Point) -> i128 { cross(sub(b, a), sub(c, b)).signum() }

/// Returns `None` if `p` and `q` don't meet.
///
/// Returns `Some(Ok([x, y]))` if `p` and `q` meet in a single point `(x, y)`.
/// Here, `x` and `y` have a type of `[i128; 2]`, where its first element is a numerator and the second element is a denominator.
///
/// Returns `Some(Err([x, y]))` if `p` and `q` overlaps at a line segment.
/// Here, `x` and `y` represent both ends of the segment.
fn intersect(p: Line, q: Line) -> Option<Result<[Frac; 2], Line>> {
	let [a, b] = sub(p[0], p[1]);
	let [c, d] = sub(q[0], q[1]);
	let u = cross([a, b], [c, d]);
	if u != 0 {
		if ccw(p[0], q[0], q[1]) * ccw(p[1], q[0], q[1]) <= 0 && ccw(q[0], p[0], p[1]) * ccw(q[1], p[0], p[1]) <= 0 {
			let [a, b, c, d] = [a, b, c, d].map(|x| x as i128);
			let [p, q] = [p, q].map(|l| l.map(|p| p.map(|x| x as i128)));
			let xn = -b * c * p[0][0] + a * (c * p[0][1] + d * q[0][0] - c * q[0][1]);
			let yn = b * (-d * p[0][0] + d * q[0][0] - c * q[0][1]) + a * d * p[0][1];
			let den = u;
			Some(Ok([[xn, den], [yn, den]]))
		} else {
			None
		}
	} else {
		if cross([a, b], sub(q[0], p[1])) != 0 || cross([a, b], sub(q[1], p[1])) != 0 {
			return None;
		}
		let (a0, a1) = (p[0].min(p[1]), p[0].max(p[1]));
		let (b0, b1) = (q[0].min(q[1]), q[0].max(q[1]));
		let (l, r) = (a0.max(b0), a1.min(b1));
		match l.cmp(&r) {
			Ordering::Less => Some(Err([l, r])),
			Ordering::Equal => Some(Ok(l.map(|x| [x as i128, 1]))),
			Ordering::Greater => None,
		}
	}
}
```

---

Last modified on 231024.