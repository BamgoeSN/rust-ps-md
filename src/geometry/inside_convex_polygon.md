# Point in a Convex Polygon

## Code
```rust,noplayground
/// Checks if point `p` is within the convex polygon `poly`.
/// Returns `None` for points on edges, `Some(true)` for inside, `Some(false)` for outside.
/// Requires `poly` to be a convex polygon oriented in CCW.
fn is_inside_convex(p: P, poly: &[P]) -> Option<bool> {
	use Ordering::*;
	if poly.len() == 1 {
		return if poly[0] == p { None } else { Some(false) };
	}
	let cmp = |a, b| angle_cmp(sub(poly[1], poly[0]))(&sub(a, poly[0]), &sub(b, poly[0]));

	let i = poly.partition_point(|&c| cmp(p, c) != Ordering::Less);
	if i == poly.len() {
		if p == poly[i - 1] {
			None
		} else {
			Some(false)
		}
	} else {
		match ccw(poly[i - 1], p, poly[i]).cmp(&0) {
			Less => Some(true),
			Equal => None,
			Greater => Some(false),
		}
	}
}
```

---

Last modified on 231225.