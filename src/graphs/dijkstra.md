# Dijkstra
`dijkstra(graph: &Graph<T>, src: usize)` where `Graph<T>` is a [graph representation](./adjlist.md), `T` is a numeric data type, and `src` is an id of a source node. It returns a `dist = Vec<Option<T>>`, where `dist[dst]` is the length of a shortest path from `src` to `dst` if it exists, or `None` if `dst` is unreachable from `src`.

## Example
```rust
#[allow(unused)]
# use std::{cmp::*, collections::*, iter, mem::*, num::*, ops::*};
# 
# fn main() {
let mut graph: Vec<Vec<(usize, i64)>> = vec![vec![]; 4];
for (u, v, w) in [(0, 1, 5), (1, 2, 5), (1, 2, 15)] {
    graph[u].push((v, w));
    graph[v].push((u, w));
}
println!("{:?}", dijkstra(&graph, 0)); // [Some(0), Some(5), Some(10), None]
# }
# 
# trait HasNz {
#     type NzType;
#     fn into_nz(self) -> Option<Self::NzType>;
#     fn retrieve(nz: Self::NzType) -> Self;
# }
# 
# macro_rules! impl_hasnz {
#     ($($t:ty, $n:ty);*) => { $(
#         impl HasNz for $t {
#             type NzType = $n;
#             fn into_nz(self) -> Option<$n> { <$n>::new(self) }
#             fn retrieve(nz: $n) -> Self { nz.get() }
#         }
#     )* };
# }
# 
# impl_hasnz!(i8, NonZeroI8; i16, NonZeroI16; i32, NonZeroI32; i64, NonZeroI64; i128, NonZeroI128; isize, NonZeroIsize);
# impl_hasnz!(u8, NonZeroU8; u16, NonZeroU16; u32, NonZeroU32; u64, NonZeroU64; u128, NonZeroU128; usize, NonZeroUsize);
# 
# fn dijkstra<T>(graph: &[Vec<(usize, T)>], src: usize) -> Vec<Option<T>>
# where
#     T: Copy + From<u8> + Add<Output = T> + Sub<Output = T> + Eq + Ord + HasNz,
#     <T as HasNz>::NzType: Copy,
# {
#     let mut dist: Vec<Option<T::NzType>> = vec![None; graph.len()];
#     let mut heap: BinaryHeap<(Reverse<T>, usize)> = BinaryHeap::new();
#     heap.push((Reverse(1.into()), src));
# 
#     while let Some((Reverse(curr_cost), curr)) = heap.pop() {
#         if dist[curr].map_or(false, |x| T::retrieve(x) < curr_cost) {
#             continue;
#         }
#         dist[curr] = curr_cost.into_nz();
# 
#         for &(next, weight) in graph[curr].iter() {
#             let next_cost = curr_cost + weight;
#             if dist[next].map_or(true, |x| T::retrieve(x) > next_cost) {
#                 dist[next] = next_cost.into_nz();
#                 heap.push((Reverse(next_cost), next));
#             }
#         }
#     }
# 
#     dist.iter().map(|x| x.map(|x| T::retrieve(x) - 1.into())).collect()
# }
```

## Code
```rust,noplayground
trait HasNz {
    type NzType;
    fn into_nz(self) -> Option<Self::NzType>;
    fn retrieve(nz: Self::NzType) -> Self;
}

macro_rules! impl_hasnz {
    ($($t:ty, $n:ty);*) => { $(
        impl HasNz for $t {
            type NzType = $n;
            fn into_nz(self) -> Option<$n> { <$n>::new(self) }
            fn retrieve(nz: $n) -> Self { nz.get() }
        }
    )* };
}

impl_hasnz!(i8, NonZeroI8; i16, NonZeroI16; i32, NonZeroI32; i64, NonZeroI64; i128, NonZeroI128; isize, NonZeroIsize);
impl_hasnz!(u8, NonZeroU8; u16, NonZeroU16; u32, NonZeroU32; u64, NonZeroU64; u128, NonZeroU128; usize, NonZeroUsize);

fn dijkstra<T>(graph: &[Vec<(usize, T)>], src: usize) -> Vec<Option<T>>
where
    T: Copy + From<u8> + Add<Output = T> + Sub<Output = T> + Eq + Ord + HasNz,
    <T as HasNz>::NzType: Copy,
{
    let mut dist: Vec<Option<T::NzType>> = vec![None; graph.len()];
    let mut heap: BinaryHeap<(Reverse<T>, usize)> = BinaryHeap::new();
    heap.push((Reverse(1.into()), src));

    while let Some((Reverse(curr_cost), curr)) = heap.pop() {
        if dist[curr].map_or(false, |x| T::retrieve(x) < curr_cost) {
            continue;
        }
        dist[curr] = curr_cost.into_nz();

        for &(next, weight) in graph[curr].iter() {
            let next_cost = curr_cost + weight;
            if dist[next].map_or(true, |x| T::retrieve(x) > next_cost) {
                dist[next] = next_cost.into_nz();
                heap.push((Reverse(next_cost), next));
            }
        }
    }

    dist.iter().map(|x| x.map(|x| T::retrieve(x) - 1.into())).collect()
}
```

### Compact Version
The code is much compact, but the performance is slightly worse than the one above. This version seems to be approximately 10% slower, but none of the definitive tests have been done to check it.

```rust,noplayground
fn dijkstra<T: Copy + From<u8> + Add<Output = T> + Eq + Ord>(graph: &[Vec<(usize, T)>], src: usize) -> Vec<Option<T>> {
    let mut dist: Vec<Option<T>> = vec![None; graph.len()];
    let mut heap: BinaryHeap<(Reverse<T>, usize)> = BinaryHeap::new();
    heap.push((Reverse(0.into()), src));

    while let Some((Reverse(curr_cost), curr)) = heap.pop() {
        if dist[curr].map_or(false, |x| x < curr_cost) {
            continue;
        }
        dist[curr] = Some(curr_cost);

        for &(next, weight) in graph[curr].iter() {
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
