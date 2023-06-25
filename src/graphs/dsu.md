# Disjoint Set Union
Disjoint set union data structure.

## Example
```rust
# fn main() {
let mut uf = UnionFind::new(10);
println!("{}", uf.find_root(2) == uf.find_root(6)); // false
uf.union(2, 6);
println!("{}", uf.find_root(2) == uf.find_root(6)); // true
# }
# 
# struct UnionFind {
#     parent: Vec<usize>,
#     size: Vec<usize>,
#     num: usize,
# }
# 
# impl UnionFind {
#     fn new(n: usize) -> Self {
#         Self {
#             parent: (0..n).collect(),
#             size: vec![1; n],
#             num: n,
#         }
#     }
# 
#     fn find_root(&mut self, mut x: usize) -> usize {
#         while self.parent[x] != x {
#             self.parent[x] = self.parent[self.parent[x]];
#             x = self.parent[x];
#         }
#         x
#     }
# 
#     fn union(&mut self, u: usize, v: usize) {
#         let u = self.find_root(u);
#         let v = self.find_root(v);
#         if u != v {
#             self.num -= 1;
#             if self.size[u] < self.size[v] {
#                 self.parent[u] = v;
#                 self.size[v] += self.size[u];
#             } else {
#                 self.parent[v] = u;
#                 self.size[u] += self.size[v];
#             }
#         }
#     }
# 
#     fn get_size(&mut self, x: usize) -> usize {
#         let r = self.find_root(x);
#         self.size[r]
#     }
# }
```

## Code

```rust,noplayground
struct UnionFind {
    parent: Vec<usize>,
    size: Vec<usize>,
    num: usize,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        Self {
            parent: (0..n).collect(),
            size: vec![1; n],
            num: n,
        }
    }

    fn find_root(&mut self, mut x: usize) -> usize {
        while self.parent[x] != x {
            self.parent[x] = self.parent[self.parent[x]];
            x = self.parent[x];
        }
        x
    }

    fn union(&mut self, u: usize, v: usize) {
        let u = self.find_root(u);
        let v = self.find_root(v);
        if u != v {
            self.num -= 1;
            if self.size[u] < self.size[v] {
                self.parent[u] = v;
                self.size[v] += self.size[u];
            } else {
                self.parent[v] = u;
                self.size[u] += self.size[v];
            }
        }
    }

    fn get_size(&mut self, x: usize) -> usize {
        let r = self.find_root(x);
        self.size[r]
    }
}
```
