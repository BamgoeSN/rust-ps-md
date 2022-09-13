# Dijkstra
`dijkstra(graph: &Graph<T>, src: usize)` where `Graph<T>` is a [graph representation](./adjlist.md), `T` is a numeric data type, and `src` is an id of a source node. It returns a `dist = Vec<Option<T>>`, where `dist[dst]` is the length of a shortest path from `src` to `dst` if it exists, or `None` if `dst` is unreachable from `src`.

## Example
```rust
# fn main() {
let mut g: Graph<i64> = Graph::new(4, 3);
g.add_edge(0, 1, 5);
g.add_edge(1, 2, 5);
g.add_edge(1, 2, 15);
println!("{:?}", dijkstra(&g, 0));
# }
# 
# #[derive(Debug)]
# struct Graph<T> {
#     n: usize,
#     first: Vec<usize>,
#     edge: Vec<(usize, usize, T)>, // (to, prev, data)
# }
# 
# impl<T> Graph<T> {
#     fn new(n: usize, e: usize) -> Self {
#         Self {
#             n,
#             first: vec![usize::MAX; n],
#             edge: Vec::with_capacity(e),
#         }
#     }
# 
#     fn add_edge(&mut self, from: usize, to: usize, data: T) {
#         let prev = std::mem::replace(&mut self.first[from], self.edge.len());
#         self.edge.push((to, prev, data));
#     }
# 
#     fn neighbor(&self, of: usize) -> Neighbor<T> {
#         Neighbor {
#             graph: self,
#             next_edge: self.first[of],
#         }
#     }
# }
# 
# struct Neighbor<'g, T> {
#     graph: &'g Graph<T>,
#     next_edge: usize,
# }
# 
# impl<'g, T> Iterator for Neighbor<'g, T> {
#     type Item = (usize, &'g T);
# 
#     fn next(&mut self) -> Option<Self::Item> {
#         let (to, next_edge, data) = self.graph.edge.get(self.next_edge)?;
#         self.next_edge = *next_edge;
#         Some((*to, data))
#     }
# }
# 
# fn dijkstra<T: Copy + From<u8> + std::ops::Add<Output = T> + Eq + Ord>(
#     graph: &Graph<T>,
#     src: usize,
# ) -> Vec<Option<T>> {
#     use std::{cmp::Reverse, collections::BinaryHeap};
#     let mut dist: Vec<Option<T>> = vec![None; graph.n];
#     let mut heap: BinaryHeap<(Reverse<T>, usize)> = BinaryHeap::new();
#     heap.push((Reverse(0.into()), src));
# 
#     while let Some((Reverse(curr_cost), curr)) = heap.pop() {
#         if dist[curr].map_or(false, |x| x < curr_cost) {
#             continue;
#         }
#         dist[curr] = Some(curr_cost);
# 
#         for (next, &weight) in graph.neighbor(curr) {
#             let next_cost = curr_cost + weight;
#             if dist[next].map_or(true, |x| x > next_cost) {
#                 dist[next] = Some(next_cost);
#                 heap.push((Reverse(next_cost), next));
#             }
#         }
#     }
#     dist
# }
```

## Code
```rust,noplayground
fn dijkstra<T: Copy + From<u8> + std::ops::Add<Output = T> + Eq + Ord>(
    graph: &Graph<T>,
    src: usize,
) -> Vec<Option<T>> {
    use std::{cmp::Reverse, collections::BinaryHeap};
    let mut dist: Vec<Option<T>> = vec![None; graph.n];
    let mut heap: BinaryHeap<(Reverse<T>, usize)> = BinaryHeap::new();
    heap.push((Reverse(0.into()), src));

    while let Some((Reverse(curr_cost), curr)) = heap.pop() {
        if dist[curr].map_or(false, |x| x < curr_cost) {
            continue;
        }
        dist[curr] = Some(curr_cost);

        for (next, &weight) in graph.neighbor(curr) {
            let next_cost = curr_cost + weight;
            if dist[next].map_or(true, |x| x > next_cost) {
                dist[next] = Some(next_cost);
                heap.push((Reverse(next_cost), next));
            }
        }
    }
    dist
}
```