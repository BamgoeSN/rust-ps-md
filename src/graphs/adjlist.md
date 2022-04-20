# Adjacency List Graph Representation

Credits to kiwiyou

```rust,noplayground
#[derive(Debug)]
struct Graph<T> {
    n: usize,
    first: Vec<usize>,
    edge: Vec<(usize, usize, T)>, // (to, prev, data)
}

impl<T> Graph<T> {
    fn new(n: usize, e: usize) -> Self {
        Self {
            n,
            first: vec![usize::MAX; n],
            edge: Vec::with_capacity(e),
        }
    }

    fn add_edge(&mut self, from: usize, to: usize, data: T) {
        let prev = std::mem::replace(&mut self.first[from], self.edge.len());
        self.edge.push((to, prev, data));
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
    next_edge: usize,
}

impl<'g, T> Iterator for Neighbor<'g, T> {
    type Item = (usize, &'g T);

    fn next(&mut self) -> Option<Self::Item> {
        let (to, next_edge, data) = self.graph.edge.get(self.next_edge)?;
        self.next_edge = *next_edge;
        Some((*to, data))
    }
}
```