# Convex Hull

`convex_hull` finds a convex hull of a given array.
It's implemented based on monotone chain algorithm, a much intuitive and straightforward convex hull algorithm compared to the well-known Graham scan.

If `COLINEAR` is set to `false`, then every point which lies on a vertex of a convex hull, but not at the endpoints of it, is excluded. If it's set to `true`, then those points are all included.

The result is sorted in clockwise direction.

If the input includes duplicates and those happen to be on the convex hull, then only one of them is included in the result.

## Code

```rust,noplayground
type Point = [i64; 2];

/// Returns a convex hull of `arr`.
/// If `COLINEAR` is set to `false`, then every point which lies on a vertex of a convex hull, but not at the endpoints of it, is excluded.
/// If `COLINEAR` is set to `true`, then such points are all included.
/// The result is sorted in clockwise direction.
///
/// The implementation utilizes monotone-chain convex hull algorithm.
fn convex_hull<const COLINEAR: bool>(arr: &[Point]) -> Vec<Point> {
	let mut arr = arr.to_owned();
	arr.sort_unstable();
	arr.dedup();
	if arr.len() <= 1 {
		return arr.to_vec();
	}
	let mut ret = vec![];

	fn monotone<const COLINEAR: bool>(it: impl Iterator<Item = Point>) -> Vec<Point> {
		let sub = |[a, b]: Point, [c, d]: Point| [a - c, b - d];
		let cross = |[a, b]: Point, [c, d]: Point| a as i128 * d as i128 - b as i128 * c as i128;
		let ccw = |a: Point, b: Point, c: Point| cross(sub(b, a), sub(c, b));
		let mut dl = vec![];
		for p in it {
			while dl.len() >= 2 {
				let n = dl.len();
				let v = ccw(dl[n - 2], dl[n - 1], p);
				if v > 0 || (!COLINEAR && v == 0) {
					dl.pop();
				} else {
					break;
				}
			}
			dl.push(p);
		}
		dl
	}

	ret.extend(monotone::<COLINEAR>(arr.iter().copied()));
	ret.pop();
	ret.extend(monotone::<COLINEAR>(arr.iter().copied().rev()));
	ret.pop();

	ret
}
```

---

Last modified on 231024.