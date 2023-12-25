# Convex Hull

`convex_hull` finds a convex hull of a given array.
It's implemented based on monotone chain algorithm, a much intuitive and straightforward convex hull algorithm compared to the well-known Graham scan.

If `COLLINEAR` is set to `false`, then every point which lies on a vertex of a convex hull, but not at the endpoints of it, is excluded. If it's set to `true`, then those points are all included.

The result is sorted in clockwise direction.

If the input includes duplicates and those happen to be on the convex hull, then only one of them is included in the result.

## Code

```rust,noplayground
/// Returns a convex hull of a set of 2D points `arr` in CCW order.
/// Set `COLLINEAR` to `true` to include, or `false` to exclude, collinear edge points.
fn convex_hull<const COLLINEAR: bool>(arr: &[P]) -> Vec<P> {
	let mut arr = arr.to_vec();
	arr.sort_unstable();
	arr.dedup();
	if arr.len() <= 1 {
		return arr.clone();
	}
	let mut ret = vec![];

	fn monotone<const COLLINEAR: bool>(it: impl Iterator<Item = P>) -> Vec<P> {
		let mut dl = vec![];
		for p in it {
			while dl.len() >= 2 {
				let n = dl.len();
				let v = ccw(dl[n - 2], dl[n - 1], p);
				if v < 0 || (!COLLINEAR && v == 0) {
					dl.pop();
				} else {
					break;
				}
			}
			dl.push(p);
		}
		dl
	}

	ret.extend(monotone::<COLLINEAR>(arr.iter().copied()));
	ret.pop();
	ret.extend(monotone::<COLLINEAR>(arr.iter().copied().rev()));
	ret.pop();

	ret
}
```

---

Last modified on 231225.