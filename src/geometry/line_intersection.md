# Line Intersection

Requires [fraction](../unclass/fraction.md) snippet.

## Code

```rust,noplayground
use frac::*;
type Point = (Frac, Frac);
type Line = (Point, Point);

fn cross(a: Point, b: Point) -> Frac {
    a.0 * b.1 - a.1 * b.0
}

fn sub(a: Point, b: Point) -> Point {
    (a.0 - b.0, a.1 - b.1)
}

#[derive(Clone, Copy, PartialEq, Eq)]
enum IntersectType {
    Separate,
    Meet(Point),
    Overlap(Line),
}
use IntersectType::*;

fn intersect(l1: Line, l2: Line) -> IntersectType {
    let (p, q) = l1;
    let (r, s) = l2;

    if p == q && r == s {
        return if p == r { Meet(p) } else { Separate };
    } else if p == q || r == s {
        let (p, q, r) = if p == q { (r, s, p) } else { (p, q, r) };
        let (px, py) = p;
        let u = sub(q, p);
        let w = sub(r, p);
        let ((ux, uy), (wx, wy)) = (u, w);
        return if cross(u, w) == 0 {
            let s = (wx * wx + wy + wy).simplify() / (ux * ux + uy * uy).simplify();
            if s >= 0 && s <= 1 {
                Meet((ux * s + px, uy * s + py))
            } else {
                Separate
            }
        } else {
            Separate
        };
    }

    let ((px, py), (rx, ry)) = (p, r);
    let u = sub(q, p);
    let v = sub(s, r);
    let w = sub(r, p);
    let ((ux, uy), (vx, vy), (wx, wy)) = (u, v, w);

    let uxv = cross(u, v);
    if uxv != 0 {
        let wxv = cross(w, v);
        let wxu = cross(w, u);
        let (a, b) = (wxv / uxv, wxu / uxv);
        return if a >= 0 && a <= 1 && b >= 0 && b <= 1 {
            let x = px + ux * a;
            let y = py + uy * a;
            Meet((x, y))
        } else {
            Separate
        };
    }

    if cross(u, w) != 0 {
        return Separate;
    }

    let gamma = if ux != 0 { vx / ux } else { vy / uy };
    let delta = if ux != 0 { wx / ux } else { wy / uy };
    let (l, r) = {
        let l = (-delta / gamma).simplify();
        let r = ((1 - delta) / gamma).simplify();
        (l.min(r), l.max(r))
    };

    if l == r {
        // Will not happen due to the debug_assertion at the beginning
        unreachable!()
    } else {
        if l > 1 || r < 0 {
            Separate
        } else {
            let lb = Frac::from(0).max(l);
            let ub = Frac::from(1).min(r);
            if lb == ub {
                let lx = rx + vx * lb;
                let ly = ry + vy * lb;
                Meet((lx, ly))
            } else {
                let lx = rx + vx * lb;
                let ly = ry + vy * lb;
                let ux = rx + vx * ub;
                let uy = ry + vy * ub;
                Overlap(((lx, ly), (ux, uy)))
            }
        }
    }
}
```

---

Last modified on 231008.