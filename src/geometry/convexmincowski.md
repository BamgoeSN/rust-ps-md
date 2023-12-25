# Mincowski Sum of Convex Polygons

## Code
```rust,noplayground
/// Returns the Minkowski sum of two convex polygons `p` and `q`.
/// Both `p` and `q` must be oriented in CCW and should not contain two equal points.
/// The returned polygon is also oriented in CCW.
fn minkowski_convex(p: &[P], q: &[P]) -> Vec<P> {
	fn line_iterator(poly: &[P]) -> impl Iterator<Item = [P; 2]> + '_ {
		let mut it = poly.iter().copied().cycle().peekable();
		iter::from_fn(move || {
			let u = it.next().unwrap();
			let v = *it.peek().unwrap();
			Some([u, v])
		})
	}

	let pi = (0..p.len()).min_by_key(|&i| p[i]).unwrap();
	let qi = (0..q.len()).min_by_key(|&i| q[i]).unwrap();
	let mut pl = line_iterator(p).skip(pi).enumerate().peekable();
	let mut ql = line_iterator(q).skip(qi).enumerate().peekable();

	let mut ret = vec![];
	while let (Some(&(pc, pp)), Some(&(qc, qq))) = (pl.peek(), ql.peek()) {
		if pc >= p.len() && qc >= q.len() {
			break;
		}
		ret.push(add(pp[0], qq[0]));
		let pcq = cross(sub(pp[1], pp[0]), sub(qq[1], qq[0]));
		if pcq >= 0 {
			pl.next();
		}
		if pcq <= 0 {
			ql.next();
		}
	}
	ret
}
```

---

Last modified on 231225.
