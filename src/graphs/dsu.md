# Disjoint Set Union
Disjoint set union (a.k.a Union-Find) processes the following queries on a graph with \\(n\\) nodes without any edges:
- Add an undirected edge between two nodes (`union`)
- Determine if there exist a path between two nodes (`is_reachable`)

```rust,noplayground
pub struct UnionFind {
    size: usize,
    parents: Vec<usize>,
    group_size: Vec<usize>,
    group_num: usize,
}

impl UnionFind {
    /// Returns a new UnionFind instance where `size` number of elements are in their own disjoint set.
    pub fn new(size: usize) -> Self {
        Self {
            size,
            parents: vec![size; size],
            group_size: vec![1; size],
            group_num: size,
        }
    }

    /// Returns the number of nodes which can be reached from x.
    pub fn get_group_size(&mut self, x: usize) -> usize {
        let root = self.find_root(x);
        self.group_size[root]
    }

    /// Returns the number of connected components.
    pub fn get_group_num(&self) -> usize {
        self.group_num
    }

    fn find_root(&mut self, x: usize) -> usize {
        if self.parents[x] == self.size {
            return x;
        }
        let root = self.find_root(self.parents[x]);
        self.parents[x] = root;
        root
    }

    /// Returns true if there exists a path from a to b.
    pub fn is_reachable(&mut self, a: usize, b: usize) -> bool {
        self.find_root(a) == self.find_root(b)
    }

    /// Add an edge between a and b.
    pub fn union(&mut self, a: usize, b: usize) {
        let a_root = self.find_root(a);
        let b_root = self.find_root(b);

        if a_root != b_root {
            self.group_num -= 1;
            let a_size = self.group_size[a_root];
            let b_size = self.group_size[b_root];
            if a_size < b_size {
                self.parents[a_root] = b_root;
                self.group_size[b_root] += a_size;
            } else {
                self.parents[b_root] = a_root;
                self.group_size[a_root] += b_size;
            }
        }
    }
}
```

## Example
```rust,noplayground
let mut uf = UnionFind::new(10);
assert_eq!(uf.is_reachable(2, 6), false);
uf.union(2, 6);
assert_eq!(uf.is_reachable(2, 6), true);
```