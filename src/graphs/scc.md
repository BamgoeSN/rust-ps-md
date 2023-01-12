# Strongly Connected Components

Finding SCCs of a directed graph. Tarjan's algorithm is used for the algorithm. [Adjancy list representation](./adjlist.md) should be included in the code for this snippet to work.

## Example
```rust
use scc::*;
# 
# fn main() {
let mut graph: Graph<()> = Graph::new(5, 5);

graph.add_edge(0, 2, ());
graph.add_edge(3, 0, ());
graph.add_edge(2, 3, ());
graph.add_edge(0, 1, ());
graph.add_edge(1, 4, ());

let scc = Scc::new(&graph);
println!("{:?}", scc.get_scc_list()); // [[3, 2, 0], [1], [4]]
println!("{:?}", scc.get_scc_ids()); // [0, 1, 0, 0, 2]
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
# mod scc {
#     use super::*;
#     struct SccStack {
#         stack: Vec<u32>,
#         check: Vec<bool>,
#     }
# 
#     impl SccStack {
#         fn new(cap: usize) -> Self {
#             Self {
#                 stack: vec![0; cap],
#                 check: vec![false; cap],
#             }
#         }
# 
#         fn push(&mut self, n: usize) {
#             self.stack.push(n as u32);
#             self.check[n] = true;
#         }
# 
#         fn pop(&mut self) -> Option<usize> {
#             let tmp = self.stack.pop()? as usize;
#             self.check[tmp] = false;
#             Some(tmp)
#         }
# 
#         fn contains(&self, n: usize) -> bool {
#             self.check[n]
#         }
#     }
# 
#     pub struct Scc<'g, T> {
#         graph: &'g Graph<T>,
#         scc_list: Vec<Vec<usize>>,
#         scc_ids: Vec<usize>,
#     }
# 
#     impl<'g, T: 'g> Scc<'g, T> {
#         pub(super) fn new(graph: &'g Graph<T>) -> Self {
#             let mut st = SccStack::new(graph.n);
#             let mut list = vec![];
#             let mut gid = 0;
#             let mut id = vec![graph.n; graph.n];
#             let mut low = vec![usize::MAX; graph.n];
# 
#             for x in 0..graph.n {
#                 if id[x] != graph.n {
#                     continue;
#                 }
#                 Self::dfs(graph, x, &mut gid, &mut id, &mut low, &mut st, &mut list);
#             }
#             list.reverse();
# 
#             let mut ids = vec![0; graph.n];
#             for (i, l) in list.iter().enumerate() {
#                 for &v in l.iter() {
#                     ids[v] = i;
#                 }
#             }
# 
#             Self {
#                 graph,
#                 scc_list: list,
#                 scc_ids: ids,
#             }
#         }
# 
#         fn dfs(
#             graph: &Graph<T>,
#             curr: usize,
#             gid: &mut usize,
#             id: &mut Vec<usize>,
#             low: &mut Vec<usize>,
#             st: &mut SccStack,
#             list: &mut Vec<Vec<usize>>,
#         ) {
#             st.push(curr);
#             id[curr] = *gid;
#             low[curr] = *gid;
#             *gid += 1;
# 
#             for (next, _) in graph.neighbor(curr) {
#                 if id[next] == graph.n {
#                     Self::dfs(graph, next, gid, id, low, st, list);
#                 }
#             }
# 
#             for (next, _) in graph.neighbor(curr) {
#                 if st.contains(next) {
#                     low[curr] = low[curr].min(low[next]);
#                 }
#             }
# 
#             if id[curr] == low[curr] {
#                 let mut newlist = vec![];
#                 while let Some(popped) = st.pop() {
#                     if popped == curr {
#                         break;
#                     }
#                     newlist.push(popped);
#                 }
#                 newlist.push(curr);
#                 list.push(newlist);
#             }
#         }
# 
#         /// Returns list of SCCs. Each vector in the list is a single SCC. The order of SCCs are
#         /// sorted in topological order. That is,, if u and v are in different SCCs, and there is
#         /// an edge from u to v, then the SCC containing u appears earlier than the SCC containing
#         /// v in the list. The order of vertices in each SCC is unspecified.
#         pub fn get_scc_list(&self) -> &Vec<Vec<usize>> {
#             &self.scc_list
#         }
# 
#         /// Returns SCC ids. If u is in scc_list[i], then scc_id[u] = i.
#         pub fn get_scc_ids(&self) -> &Vec<usize> {
#             &self.scc_ids
#         }
#     }
# }
```

## Code
```rust,noplayground
mod scc {
    use super::*;
    struct SccStack {
        stack: Vec<u32>,
        check: Vec<bool>,
    }

    impl SccStack {
        fn new(cap: usize) -> Self {
            Self {
                stack: vec![0; cap],
                check: vec![false; cap],
            }
        }

        fn push(&mut self, n: u32) {
            self.stack.push(n);
            self.check[n as usize] = true;
        }

        fn pop(&mut self) -> Option<u32> {
            let tmp = self.stack.pop()?;
            self.check[tmp as usize] = false;
            Some(tmp)
        }

        fn contains(&self, n: u32) -> bool {
            self.check[n as usize]
        }
    }

    pub struct Scc<'g, T> {
        graph: &'g Graph<T>,
        scc_list: Vec<Vec<usize>>,
        scc_ids: Vec<usize>,
    }

    impl<'g, T: 'g> Scc<'g, T> {
        pub(super) fn new(graph: &'g Graph<T>) -> Self {
            let mut st = SccStack::new(graph.n);
            let mut list = vec![];
            let mut gid = 0;
            let mut id = vec![graph.n as u32; graph.n];
            let mut low = vec![u32::MAX; graph.n];

            for x in 0..graph.n as u32 {
                if id[x as usize] != graph.n as u32 {
                    continue;
                }
                Self::dfs(graph, x, &mut gid, &mut id, &mut low, &mut st, &mut list);
            }
            list.reverse();

            let mut ids = vec![0; graph.n];
            for (i, l) in list.iter().enumerate() {
                for &v in l.iter() {
                    ids[v] = i;
                }
            }

            Self {
                graph,
                scc_list: list,
                scc_ids: ids,
            }
        }

        fn dfs(
            graph: &Graph<T>,
            curr: u32,
            gid: &mut u32,
            id: &mut Vec<u32>,
            low: &mut Vec<u32>,
            st: &mut SccStack,
            list: &mut Vec<Vec<usize>>,
        ) {
            st.push(curr);
            id[curr as usize] = *gid;
            low[curr as usize] = *gid;
            *gid += 1;

            for (next, _) in graph.neighbor(curr as usize) {
                if id[next] == graph.n as u32 {
                    Self::dfs(graph, next as u32, gid, id, low, st, list);
                }
            }

            for (next, _) in graph.neighbor(curr as usize) {
                if st.contains(next as u32) {
                    low[curr as usize] = low[curr as usize].min(low[next]);
                }
            }

            if id[curr as usize] == low[curr as usize] {
                let mut newlist = vec![];
                while let Some(popped) = st.pop() {
                    if popped == curr {
                        break;
                    }
                    newlist.push(popped as usize);
                }
                newlist.push(curr as usize);
                list.push(newlist);
            }
        }

        /// Returns list of SCCs. Each vector in the list is a single SCC. The order of SCCs are
        /// sorted in topological order. That is,, if u and v are in different SCCs, and there is
        /// an edge from u to v, then the SCC containing u appears earlier than the SCC containing
        /// v in the list. The order of vertices in each SCC is unspecified.
        pub fn get_scc_list(&self) -> &Vec<Vec<usize>> {
            &self.scc_list
        }

        /// Returns SCC ids. If u is in scc_list[i], then scc_id[u] = i.
        pub fn get_scc_ids(&self) -> &Vec<usize> {
            &self.scc_ids
        }
    }
}
```
