# Convex Hull

`convex_hull` finds a convex hull of a given array.
It's implemented based on monotone chain algorithm, a much intuitive and straightforward convex hull algorithm compared to the well-known Graham scan.

If `COLLINEAR` is set to `false`, then every point which lies on a vertex of a convex hull, but not at the endpoints of it, is excluded. If it's set to `true`, then those points are all included.

The result is sorted in clockwise direction.

If the input includes duplicates and those happen to be on the convex hull, then only one of them is included in the result.

## Code

```rust,noplayground
type I = i64;
type P = [I; 2];

fn sub(a: P, b: P) -> P { [a[0] - b[0], a[1] - b[1]] }
fn cross(a: P, b: P) -> I { a[0] * b[1] - a[1] * b[0] }
fn ccw(a: P, b: P, c: P) -> I { cross(sub(b, a), sub(c, b)) }

/// Returns a convex hull of `arr`.
/// If `COLLINEAR` is set to `false`, then every point which lies on a vertex of a convex hull, but not at the endpoints of it, is excluded.
/// If `COLLINEAR` is set to `true`, then such points are all included.
/// The result is sorted in CCW direction.
///
/// The implementation utilizes monotone-chain convex hull algorithm.
fn convex_hull<const COLLINEAR: bool>(arr: &[P]) -> Vec<P> {
	let mut arr = arr.to_owned();
	arr.sort_unstable();
	arr.dedup();
	if arr.len() <= 1 {
		return arr;
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

Last modified on 231203.