# Farthest Points

## Code
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