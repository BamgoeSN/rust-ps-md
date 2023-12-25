# Point in a Polygon

## Code
```rust,noplayground
/// Checks if point `p` is within the polygon `poly`.
/// Returns `None` for points on edges, `Some(true)` for inside, and `Some(false)` for outside.
/// Requires `poly` to be a non-self-intersecting polygon. Orientation does not matter.
fn is_inside(p: P, poly: &[P]) -> Option<bool> {
	let n = poly.len();
	let mut it = poly.iter().copied().cycle().peekable();
	let mut nxt = || [it.next().unwrap(), *it.peek().unwrap()];

	for l in (0..n).map(|_| nxt()) {
		if meets(l, [p, p]) {
			return None;
		}
	}

	let cnt = (0..n)
		.map(|_| nxt())
		.filter(|&l| {
			let half = (l[0][1] < p[1]) != (l[1][1] < p[1]);
			let touch = meets(l, [p, [p[0].max(l[0][0]).max(l[1][0]), p[1]]]);
			half && touch
		})
		.count();
	Some(cnt % 2 == 1)
}
```

---

Last modified on 231225.