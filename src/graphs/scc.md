# Strongly Connected Components

Finding SCCs of a directed graph. Tarjan's algorithm is used for the algorithm.

## Example
```rust
use scc::*;

# fn main() {
let mut graph = vec![vec![]; 5];
for (u, v) in [(0, 2), (3, 0), (2, 3), (0, 1), (1, 4)] {
    graph[u].push(v);
}

let scc_list = find_scc(&graph);
println!("{:?}", scc_list); // [[3, 2, 0], [1], [4]]
let scc_id = gen_scc_ids(&graph, &scc_list);
println!("{:?}", scc_id); // [0, 1, 0, 0, 2]
let scc_graph = gen_scc_graph(&graph, &scc_list, &scc_id);
println!("{:?}", scc_graph); // [[1], [2], []]
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
```

## Code
```rust,noplayground
mod scc {
    struct SccStack {
        stack: Vec<u32>,
        check: Vec<u64>,
    }

    impl SccStack {
        fn new(cap: usize) -> Self {
            Self {
                stack: vec![0; cap],
                check: vec![0; (cap + 63) / 64],
            }
        }
        fn push(&mut self, n: usize) {
            self.stack.push(n as u32);
            self.check[n / 64] |= 1 << (n % 64);
        }
        fn pop(&mut self) -> Option<usize> {
            let tmp = self.stack.pop()? as usize;
            self.check[tmp / 64] &= !(1 << (tmp % 64));
            Some(tmp)
        }
        fn contains(&self, n: usize) -> bool {
            self.check[n / 64] & (1 << (n % 64)) != 0
        }
    }

    struct DfsPack {
        gid: usize,
        id: Vec<usize>,
        low: Vec<usize>,
        st: SccStack,
    }

    fn dfs(n: usize, graph: &[Vec<usize>], curr: usize, p: &mut DfsPack, list: &mut Vec<Vec<usize>>) {
        p.st.push(curr);
        p.id[curr] = p.gid;
        p.low[curr] = p.gid;
        p.gid += 1;

        for &next in graph[curr].iter() {
            if p.id[next] == n {
                dfs(n, graph, next, p, list);
            }
        }
        for &next in graph[curr].iter() {
            if p.st.contains(next) {
                p.low[curr] = p.low[curr].min(p.low[next]);
            }
        }

        if p.id[curr] == p.low[curr] {
            let mut newlist = vec![];
            while let Some(popped) = p.st.pop() {
                if popped == curr {
                    break;
                }
                newlist.push(popped);
            }
            newlist.push(curr);
            list.push(newlist);
        }
    }

    /// Returns a list of SCCs of `graph`.
    /// The returned list is a 2D vector of `usize`, which consists of a list of vertices within a same SCC.
    /// The order of the SCCs in the returned list is topologically sorted.
    ///
    /// The implementation uses Tarjan's SCC algorithm.
    pub fn find_scc(graph: &[Vec<usize>]) -> Vec<Vec<usize>> {
        let n = graph.len();
        let mut list = vec![];

        let mut p = DfsPack {
            gid: 0,
            id: vec![n; n],
            low: vec![usize::MAX; n],
            st: SccStack::new(n),
        };
        for x in 0..n {
            if p.id[x] != n {
                continue;
            }
            dfs(n, graph, x, &mut p, &mut list);
        }
        list.reverse();
        list
    }

    /// Returns a list about what SCC each vertices are in.
    /// `scc_list` has to be generated in advance from `find_scc`.
    pub fn gen_scc_ids(graph: &[Vec<usize>], scc_list: &[Vec<usize>]) -> Vec<usize> {
        let mut ids = vec![0; graph.len()];
        for (i, l) in scc_list.iter().enumerate() {
            for &v in l {
                ids[v] = i;
            }
        }
        ids
    }

    /// Returns a graph of SCCs. The number of vertices of the new graph will be the number of SCCs in the graph.
    /// `scc_list` and `scc_ids` have to be generated in advanced from `find_scc` and `gen_scc_ids`.
    pub fn gen_scc_graph(graph: &[Vec<usize>], scc_list: &[Vec<usize>], scc_ids: &[usize]) -> Vec<Vec<usize>> {
        let mut ret = vec![vec![]; scc_list.len()];
        for u in 0..graph.len() {
            let a = scc_ids[u];
            for &v in graph[u].iter() {
                let b = scc_ids[v];
                if a < b {
                    ret[a].push(b);
                }
            }
        }
        ret
    }
}
```

---

Last updated on 231008.