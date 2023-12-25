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

The code below only checks if two lines meet, without calculating where they meet.

```rust,noplayground
/// Returns the two farthest points from `p`, or `None` if `p` is empty.
fn farthest_points(p: &[P]) -> Option<[P; 2]> {
	let dist = |a: P, b: P| dot(sub(a, b), sub(a, b));

	let cvh = convex_hull::<false>(p);
	if cvh.len() == 1 {
		return Some([cvh[0], cvh[0]]);
	} else if cvh.len() == 2 {
		return Some([cvh[0], cvh[1]]);
	}

	fn line_iterator(poly: &[P]) -> impl Iterator<Item = [P; 2]> + '_ {
		let mut it = poly.iter().copied().cycle().peekable();
		iter::from_fn(move || {
			let u = it.next().unwrap();
			let v = *it.peek().unwrap();
			Some([u, v])
		})
	}

	let mut r = line_iterator(&cvh).skip(1).peekable();
	let mut rec = None;
	for [la, lb] in line_iterator(&cvh).take(cvh.len()) {
		let lv = sub(lb, la);
		while {
			let &[ra, rb] = r.peek().unwrap();
			cross(lv, sub(rb, ra)) > 0
		} {
			r.next();
		}
		let &[ra, _] = r.peek().unwrap();
		let ch = [la, ra];
		let d = dist(ch[0], ch[1]);
		if rec.map_or(true, |(x, _)| x < d) {
			rec = Some((d, ch));
		}
	}

	rec.map(|x| x.1)
}
```

---

Last modified on 231225.