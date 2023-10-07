# Adjacency List Graph Representation

Deprecated because it's way easier to use `Vec<Vec<(usize, T)>>`. The performance gain isn't worth giving up usability.

Credits to kiwiyou

```rust,noplayground
#[derive(Debug)]
struct Graph<T> {
    n: usize,
    first: Vec<u32>,
    edge: Vec<(u32, u32, T)>, // (to, prev, data)
}

impl<T> Graph<T> {
    fn new(n: usize, e: usize) -> Self {
        Self {
            n,
            first: vec![u32::MAX; n],
            edge: Vec::with_capacity(e),
        }
    }

    fn add_edge(&mut self, from: usize, to: usize, data: T) {
        let prev = std::mem::replace(&mut self.first[from], self.edge.len() as u32);
        self.edge.push((to as u32, prev, data));
    }

    fn neighbor(&self, of: usize) -> Neighbor<T> {
        Neighbor {
            graph: self,
            next_edge: self.first[of],
        }
    }
}

struct Neighbor<'g, T> {
    graph: &'g Graph<T>,
    next_edge: u32,
}

impl<'g, T> Iterator for Neighbor<'g, T> {
    type Item = (usize, &'g T);

    fn next(&mut self) -> Option<Self::Item> {
        let (to, next_edge, data) = self.graph.edge.get(self.next_edge as usize)?;
        self.next_edge = *next_edge;
        Some((*to as usize, data))
    }
}
```

---

Last updated on 231008.