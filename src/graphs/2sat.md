# 2-SAT

2-SAT with \\(N\\) clauses can be solved with time complexity of \\(O(N)\\). SCC module should be with this snippet in the code.

## Example

```rust
use twosat::*;

# 
# fn main() {
// (not 0 or 1) and (not 1 or 2) and (0 or 2) and (2 or 1)
let mut ts = TwoSat::new(3, 4);
ts.add_clause(0, false, 1, true);
ts.add_clause(1, false, 2, true);
ts.add_clause(0, true, 2, true);
ts.add_clause(2, true, 1, true);
println!("{:?}", ts.solve()); // Some([false, true, true])

// (0 or 0) and (not 0 or not 0)
let mut ts = TwoSat::new(1, 2);
ts.add_clause(0, true, 0, true);
ts.add_clause(0, false, 0, false);
println!("{:?}", ts.solve()); // None
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
# 
# mod twosat {
#     use super::{scc::*, *};
# 
#     pub struct TwoSat {
#         n: usize,
#         graph: Graph<()>,
#     }
# 
#     impl TwoSat {
#         pub fn new(n: usize, clause_num: usize) -> Self {
#             Self {
#                 n,
#                 graph: Graph::new(n << 1, clause_num << 1),
#             }
#         }
# 
#         fn judge(f: bool, a: usize, b: usize) -> usize {
#             if f {
#                 a
#             } else {
#                 b
#             }
#         }
# 
#         /// Adds a clause ("not" if !f)i or ("not" if !g)j.
#         pub fn add_clause(&mut self, i: usize, f: bool, j: usize, g: bool) {
#             self.graph.add_edge(
#                 (i << 1) + Self::judge(f, 0, 1),
#                 (j << 1) + Self::judge(g, 1, 0),
#                 (),
#             );
#             self.graph.add_edge(
#                 (j << 1) + Self::judge(g, 0, 1),
#                 (i << 1) + Self::judge(f, 1, 0),
#                 (),
#             );
#         }
# 
#         pub fn solve(self) -> Option<Vec<bool>> {
#             let mut answer = vec![false; self.n];
# 
#             let scc = Scc::new(&self.graph);
#             let ids = scc.get_scc_ids();
# 
#             for i in 0..self.n {
#                 if ids[i << 1] == ids[(i << 1) + 1] {
#                     return None;
#                 }
#                 answer[i] = ids[i << 1] < ids[(i << 1) + 1];
#             }
# 
#             Some(answer)
#         }
#     }
# }
```









## Code

```rust,noplayground
mod twosat {
    use super::{scc::*, *};

    pub struct TwoSat {
        n: usize,
        graph: Graph<()>,
    }

    impl TwoSat {
        pub fn new(n: usize, clause_num: usize) -> Self {
            Self {
                n,
                graph: Graph::new(n << 1, clause_num << 1),
            }
        }

        fn judge(f: bool, a: usize, b: usize) -> usize {
            if f {
                a
            } else {
                b
            }
        }

        /// Adds a clause ("not" if !f)i or ("not" if !g)j.
        pub fn add_clause(&mut self, i: usize, f: bool, j: usize, g: bool) {
            self.graph.add_edge(
                (i << 1) + Self::judge(f, 0, 1),
                (j << 1) + Self::judge(g, 1, 0),
                (),
            );
            self.graph.add_edge(
                (j << 1) + Self::judge(g, 0, 1),
                (i << 1) + Self::judge(f, 1, 0),
                (),
            );
        }

        pub fn solve(self) -> Option<Vec<bool>> {
            let mut answer = vec![false; self.n];

            let scc = Scc::new(&self.graph);
            let ids = scc.get_scc_ids();

            for i in 0..self.n {
                if ids[i << 1] == ids[(i << 1) + 1] {
                    return None;
                }
                answer[i] = ids[i << 1] < ids[(i << 1) + 1];
            }

            Some(answer)
        }
    }
}
```
