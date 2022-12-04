# Disjoint Set Union
Disjoint set union data structure.

## Example
```rust
# struct UnionFind {
#     n: usize,
#     parent: Vec<usize>,
#     group_num: usize,
#     group_size: Vec<usize>,
# }
# impl UnionFind {
#     fn new(n: usize) -> Self {
#         Self {
#             n,
#             parent: vec![n; n],
#             group_num: n,
#             group_size: vec![1; n],
#         }
#     }
#     fn find(&mut self, u: usize) -> usize {
#         let par = self.parent[u];
#         if par == self.n {
#             return u;
#         }
#         self.parent[u] = self.find(par);
#         self.parent[u]
#     }
#     fn is_differ(&mut self, u: usize, v: usize) -> bool {
#         self.find(u) != self.find(v)
#     }
#     fn union(&mut self, u: usize, v: usize) {
#         let (ur, vr) = (self.find(u), self.find(v));
#         if ur != vr {
#             self.parent[vr] = ur;
#             self.group_size[ur] += self.group_size[vr];
#             self.group_num -= 1;
#         }
#     }
#     fn get_group_size(&mut self, u: usize) -> usize {
#         let r = self.find(u);
#         self.group_size[r]
#     }
# }
# fn main() {
let mut uf = UnionFind::new(10);
println!("{}", uf.is_differ(2, 6)); // true
uf.union(2, 6);
println!("{}", uf.is_differ(2, 6)); // false
# }
```

## Code

```rust,noplayground
struct UnionFind {
    n: usize,
    parent: Vec<usize>,
    group_num: usize,
    group_size: Vec<usize>,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        Self {
            n,
            parent: vec![n; n],
            group_num: n,
            group_size: vec![1; n],
        }
    }

    /// Returns the root of a group u is in
    fn find(&mut self, u: usize) -> usize {
        let par = self.parent[u];
        if par == self.n {
            return u;
        }
        self.parent[u] = self.find(par);
        self.parent[u]
    }

    /// Returns true if u and v are in different groups, false otherwise.
    fn is_differ(&mut self, u: usize, v: usize) -> bool {
        self.find(u) != self.find(v)
    }

    /// The group of v is merged into the group of u
    fn union(&mut self, u: usize, v: usize) {
        let (ur, vr) = (self.find(u), self.find(v));
        if ur != vr {
            self.parent[vr] = ur;
            self.group_size[ur] += self.group_size[vr];
            self.group_num -= 1;
        }
    }

    fn get_group_size(&mut self, u: usize) -> usize {
        let r = self.find(u);
        self.group_size[r]
    }
}
```