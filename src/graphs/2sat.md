# 2-SAT

2-SAT with \\(N\\) clauses can be solved with time complexity of \\(O(N)\\). [SCC](./scc.md) should be with this snippet in the code.

## Example

```rust
use twosat::*;

# fn main() {
// (not 0 or 1) and (not 1 or 2) and (0 or 2) and (2 or 1)
let mut ts = TwoSat::new(3);
for (a, b) in [((0, false), (1, true)), ((1, false), (2, true)), ((0, true), (2, true)), ((2, true), (1, true))] {
    ts.add_clause(a, b);
}
println!("{:?}", ts.solve()); // Some([false, true, true])

// (0 or 0) and (not 0 or not 0)
let mut ts = TwoSat::new(1);
for (a, b) in [((0, true), (0, true)), ((0, false), (0, false))] {
    ts.add_clause(a, b);
}
println!("{:?}", ts.solve()); // None
# }
# 
# mod scc {
#     struct SccStack {
#         stack: Vec<u32>,
#         check: Vec<u64>,
#     }
# 
#     impl SccStack {
#         fn new(cap: usize) -> Self {
#             Self {
#                 stack: vec![0; cap],
#                 check: vec![0; (cap + 63) / 64],
#             }
#         }
#         fn push(&mut self, n: usize) {
#             self.stack.push(n as u32);
#             self.check[n / 64] |= 1 << (n % 64);
#         }
#         fn pop(&mut self) -> Option<usize> {
#             let tmp = self.stack.pop()? as usize;
#             self.check[tmp / 64] &= !(1 << (tmp % 64));
#             Some(tmp)
#         }
#         fn contains(&self, n: usize) -> bool {
#             self.check[n / 64] & (1 << (n % 64)) != 0
#         }
#     }
# 
#     struct DfsPack {
#         gid: usize,
#         id: Vec<usize>,
#         low: Vec<usize>,
#         st: SccStack,
#     }
# 
#     fn dfs(n: usize, graph: &[Vec<usize>], curr: usize, p: &mut DfsPack, list: &mut Vec<Vec<usize>>) {
#         p.st.push(curr);
#         p.id[curr] = p.gid;
#         p.low[curr] = p.gid;
#         p.gid += 1;
# 
#         for &next in graph[curr].iter() {
#             if p.id[next] == n {
#                 dfs(n, graph, next, p, list);
#             }
#         }
#         for &next in graph[curr].iter() {
#             if p.st.contains(next) {
#                 p.low[curr] = p.low[curr].min(p.low[next]);
#             }
#         }
# 
#         if p.id[curr] == p.low[curr] {
#             let mut newlist = vec![];
#             while let Some(popped) = p.st.pop() {
#                 if popped == curr {
#                     break;
#                 }
#                 newlist.push(popped);
#             }
#             newlist.push(curr);
#             list.push(newlist);
#         }
#     }
# 
#     /// Returns a list of SCCs of `graph`.
#     /// The returned list is a 2D vector of `usize`, which consists of a list of vertices within a same SCC.
#     /// The order of the SCCs in the returned list is topologically sorted.
#     ///
#     /// The implementation uses Tarjan's SCC algorithm.
#     pub fn find_scc(graph: &[Vec<usize>]) -> Vec<Vec<usize>> {
#         let n = graph.len();
#         let mut list = vec![];
# 
#         let mut p = DfsPack {
#             gid: 0,
#             id: vec![n; n],
#             low: vec![usize::MAX; n],
#             st: SccStack::new(n),
#         };
#         for x in 0..n {
#             if p.id[x] != n {
#                 continue;
#             }
#             dfs(n, graph, x, &mut p, &mut list);
#         }
#         list.reverse();
#         list
#     }
# 
#     /// Returns a list about what SCC each vertices are in.
#     /// `scc_list` has to be generated in advance from `find_scc`.
#     pub fn gen_scc_ids(graph: &[Vec<usize>], scc_list: &[Vec<usize>]) -> Vec<usize> {
#         let mut ids = vec![0; graph.len()];
#         for (i, l) in scc_list.iter().enumerate() {
#             for &v in l {
#                 ids[v] = i;
#             }
#         }
#         ids
#     }
# 
#     /// Returns a graph of SCCs. The number of vertices of the new graph will be the number of SCCs in the graph.
#     /// `scc_list` and `scc_ids` have to be generated in advanced from `find_scc` and `gen_scc_ids`.
#     pub fn gen_scc_graph(graph: &[Vec<usize>], scc_list: &[Vec<usize>], scc_ids: &[usize]) -> Vec<Vec<usize>> {
#         let mut ret = vec![vec![]; scc_list.len()];
#         for u in 0..graph.len() {
#             let a = scc_ids[u];
#             for &v in graph[u].iter() {
#                 let b = scc_ids[v];
#                 if a < b {
#                     ret[a].push(b);
#                 }
#             }
#         }
#         ret
#     }
# }
# 
# mod twosat {
#     use super::scc::*;
# 
#     /// 2-SAT solver.
#     pub struct TwoSat {
#         n: usize,
#         graph: Vec<Vec<usize>>,
#     }
# 
#     impl TwoSat {
#         /// Creates a new instance of 2-SAT solver.
#         pub fn new(n: usize) -> Self {
#             Self { n, graph: vec![vec![]; n << 1] }
#         }
# 
#         /// Adds a clause of `(i, f) & (j, g)`.
#         /// For example, `self.add_clause((0, false), (1, true))` is adding a clause `~x0 & x1` to the solver.
#         pub fn add_clause(&mut self, (i, f): (usize, bool), (j, g): (usize, bool)) {
#             let judge = |x: bool, a: usize, b: usize| if x { a } else { b };
#             self.graph[i * 2 + judge(f, 0, 1)].push(j * 2 + judge(g, 1, 0));
#             self.graph[j * 2 + judge(g, 0, 1)].push(i * 2 + judge(f, 1, 0));
#         }
# 
#         /// Returns any possible solution of the 2-SAT problem if there's any in O(N) time.
#         /// Returns `None` if the problem is unsolvable.
#         pub fn solve(&self) -> Option<Vec<bool>> {
#             let mut ans = vec![false; self.n];
#             let scc_list = find_scc(&self.graph);
#             let ids = gen_scc_ids(&self.graph, &scc_list);
#             for i in 0..self.n {
#                 if ids[i * 2] == ids[i * 2 + 1] {
#                     return None;
#                 }
#                 ans[i] = ids[i * 2] < ids[i * 2 + 1];
#             }
#             Some(ans)
#         }
#     }
# }
```

## Code

```rust,noplayground
mod twosat {
    use super::scc::*;

    /// 2-SAT solver.
    pub struct TwoSat {
        n: usize,
        graph: Vec<Vec<usize>>,
    }

    impl TwoSat {
        /// Creates a new instance of 2-SAT solver.
        pub fn new(n: usize) -> Self {
            Self { n, graph: vec![vec![]; n << 1] }
        }

        /// Adds a clause of `(i, f) & (j, g)`.
        /// For example, `self.add_clause((0, false), (1, true))` is adding a clause `~x0 & x1` to the solver.
        pub fn add_clause(&mut self, (i, f): (usize, bool), (j, g): (usize, bool)) {
            let judge = |x: bool, a: usize, b: usize| if x { a } else { b };
            self.graph[i * 2 + judge(f, 0, 1)].push(j * 2 + judge(g, 1, 0));
            self.graph[j * 2 + judge(g, 0, 1)].push(i * 2 + judge(f, 1, 0));
        }

        /// Returns any possible solution of the 2-SAT problem if there's any in O(N) time.
        /// Returns `None` if the problem is unsolvable.
        pub fn solve(&self) -> Option<Vec<bool>> {
            let mut ans = vec![false; self.n];
            let scc_list = find_scc(&self.graph);
            let ids = gen_scc_ids(&self.graph, &scc_list);
            for i in 0..self.n {
                if ids[i * 2] == ids[i * 2 + 1] {
                    return None;
                }
                ans[i] = ids[i * 2] < ids[i * 2 + 1];
            }
            Some(ans)
        }
    }
}
```

---

Last updated on 231008.