# Convex Hull

`cvh(pts: &[Point])` returns indices of points included in a convex hull of `pts`.
It's implemented based on monotone chain algorithm, a much intuitive and straightforward convex hull
algorithm compared to the well-known Graham scan.

The code below excludes the points on the sides of the hull, but by simply changing `>=` from lines calling `ccw` to `>`,
one can change its behavior to include such points.

If `pts` includes duplicates and those happen to be on the convex hull, then only one of them is included in the result.

## Code

```rust,noplayground
type Coord = i32;
type CCWOut = i64;
type Point = (Coord, Coord);

fn ccw(a: &Point, b: &Point, c: &Point) -> CCWOut {
    let ba = (b.0 as CCWOut - a.0 as CCWOut, b.1 as CCWOut - a.1 as CCWOut);
    let cb = (c.0 as CCWOut - b.0 as CCWOut, c.1 as CCWOut - b.1 as CCWOut);
    ba.0 * cb.1 - ba.1 * cb.0
}

fn cvh(pts: &[Point]) -> Vec<usize> {
    if pts.is_empty() {
        return vec![];
    } else if pts.len() == 1 {
        return vec![0];
    }

    let n = pts.len();

    let mut refs: Vec<_> = (0..n).collect();
    refs.sort_unstable_by_key(|&i| pts[i]);
    refs.dedup_by_key(|i| pts[*i]);

    let mut upper = vec![refs[0]];
    for &pt in refs.iter().skip(1) {
        while upper.len() > 1 {
            let ul = upper.len();
            if ccw(&pts[upper[ul - 2]], &pts[upper[ul - 1]], &pts[pt]) > 0 {
                upper.pop();
            } else {
                break;
            }
        }
        upper.push(pt);
    }

    let mut lower = vec![*refs.last().unwrap()];
    for &pt in refs.iter().rev().skip(1) {
        while lower.len() > 1 {
            let ll = lower.len();
            if ccw(&pts[lower[ll - 2]], &pts[lower[ll - 1]], &pts[pt]) > 0 {
                lower.pop();
            } else {
                break;
            }
        }
        lower.push(pt);
    }

    lower.pop();
    upper.into_iter().chain(lower.into_iter().skip(1)).collect()
}
```
