# Angle Comparator

## Code
```rust,noplayground
/// Returns a comparator for points relative to `x_axis`: first by CCW angle, then by distance from the origin for equal angles.
/// The origin is considered the smallest.
fn angle_cmp(x_axis: P) -> impl Fn(&P, &P) -> Ordering {
	move |&a, &b| {
		let ud = |p| cross(p, x_axis) > 0 || (cross(p, x_axis) == 0 && dot(p, x_axis) < 0);
		ud(a).cmp(&ud(b)).then_with(|| cross(b, a).cmp(&0).then_with(|| dot(a, a).cmp(&dot(b, b))))
	}
}
```

---

Last modified on 231225.