# Weighted DSU

Disjoint set union where vertices have their own "potential value".

## Code
```rust,noplayground
struct WeightDSU {
    n: usize,
    group_num: usize,
    parent: Vec<usize>,
    group_size: Vec<usize>,
    potdiff: Vec<i64>,
}

impl WeightDSU {
    fn new(n: usize) -> Self {
        Self {
            n,
            group_num: n,
            parent: vec![n; n],
            group_size: vec![1; n],
            potdiff: vec![0; n],
        }
    }

    /// Returns the root of u, and pot[u]-pot[root]
    fn find(&mut self, u: usize) -> (usize, i64) {
        let par = self.parent[u];
        if par == self.n {
            return (u, 0);
        }
        let (root, parpoten) = self.find(par);
        self.potdiff[u] += parpoten;
        self.parent[u] = root;
        (root, self.potdiff[u])
    }

    /// Returns pot[u]-pot[v]. Returns None if u and v aren't in the same group.
    fn pot_differ(&mut self, u: usize, v: usize) -> Option<i64> {
        let (ur, udiff) = self.find(u);
        let (vr, vdiff) = self.find(v);
        if ur != vr {
            None
        } else {
            Some(udiff - vdiff)
        }
    }

    /// Unions u and v, with setting pot[u]-pot[v] = w.
    /// Also puts the root of v under the root of u.
    /// Returns false if u and v are already unioned and the relation is not consistent.
    fn union(&mut self, u: usize, v: usize, w: i64) -> bool {
        let (ur, udiff) = self.find(u);
        let (vr, vdiff) = self.find(v);
        if ur == vr {
            udiff - vdiff == w
        } else {
            self.parent[vr] = ur;
            self.potdiff[vr] = udiff - vdiff - w;
            self.group_size[ur] += self.group_size[vr];
            self.group_num -= 1;
            true
        }
    }

    fn get_group_size(&mut self, u: usize) -> usize {
        let r = self.find(u).0;
        self.group_size[r]
    }
}
```