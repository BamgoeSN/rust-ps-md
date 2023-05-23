# Dinic's Algorithm

Dinic's algorithm is a practically fast algorithm for computing the maximum flow in a flow network.

User can create a network instance with `Dinic::new(n)` where `n` is the number of vertices in the network,
and add edges with `Dinic::add_edges(&mut self, src, dst, cap)` method where `cap` is the capacity of the edge being added.

Calling `Dinic::max_flow(&mut self, src, dst)` calculates the maximum flow from `src` to `dst`.
After the calculation, remaining capacity of any edges can be found by directly inspecting into `cap` field of a wanted edge.

## Example

The example below calculates maximum flow of a flow network shown on page 5 of <https://github.com/justiceHui/SSU-SCCC-Study/blob/master/2022-winter-adv/slide/02.pdf>.
 
```rust
# use dinic::Dinic;
# fn main() {
let mut dn = Dinic::new(4);
for (src, dst, cap) in [(0, 1, 2), (0, 2, 2), (1, 2, 1), (1, 3, 2), (2, 3, 2)] {
    dn.add_edge(src, dst, cap);
}

let (src, dst) = (0, 3);
let max_flow = dn.max_flow(src, dst);
println!("{max_flow}"); // 4

// Remaining capacity of an edge from 0 to 1
let edge = dn.g[0].iter().filter(|e| e.dst == 1).next().unwrap();
println!("{}", edge.cap); // 0
# }
# 
# mod dinic {
#     //! Reference: https://github.com/justiceHui/SSU-SCCC-Study/blob/master/2022-winter-adv/slide/04.pdf
# 
#     use std::collections::VecDeque;
# 
#     #[derive(Clone)]
#     pub struct Edge {
#         pub dst: u32,
#         pub opp: u32,
#         pub cap: u64,
#     }
# 
#     impl Edge {
#         fn new(dst: usize, opp: usize, cap: u64) -> Self {
#             Self {
#                 dst: dst as u32,
#                 opp: opp as u32,
#                 cap,
#             }
#         }
#     }
# 
#     pub struct Dinic {
#         pub n: usize,
#         pub g: Vec<Vec<Edge>>,
#     }
# 
#     impl Dinic {
#         pub fn new(n: usize) -> Self {
#             Self {
#                 n,
#                 g: vec![vec![]; n],
#             }
#         }
# 
#         pub fn add_edge(&mut self, s: usize, e: usize, cap: u64) {
#             let sl = self.g[s].len();
#             let el = self.g[e].len();
#             self.g[s].push(Edge::new(e, el, cap));
#             self.g[e].push(Edge::new(s, sl, 0));
#         }
# 
#         fn bfs(&mut self, s: u32, t: u32, lv: &mut [u32]) -> bool {
#             lv.fill(0);
# 
#             let mut queue = VecDeque::new();
#             queue.push_back(s);
#             lv[s as usize] = 1;
# 
#             while let Some(v) = queue.pop_front() {
#                 for e in self.g[v as usize].iter() {
#                     if lv[e.dst as usize] == 0 && e.cap != 0 {
#                         queue.push_back(e.dst);
#                         lv[e.dst as usize] = lv[v as usize] + 1;
#                     }
#                 }
#             }
# 
#             lv[t as usize] != 0
#         }
# 
#         fn dfs(&mut self, v: u32, t: u32, fl: u64, lv: &[u32], idx: &mut [u32]) -> u64 {
#             if v == t || fl == 0 {
#                 return fl;
#             }
# 
#             for i in idx[v as usize]..self.g[v as usize].len() as u32 {
#                 idx[v as usize] = i;
# 
#                 let Edge { dst, opp, cap } = self.g[v as usize][i as usize];
#                 if lv[dst as usize] != lv[v as usize] + 1 || cap == 0 {
#                     continue;
#                 }
#                 let now = self.dfs(dst, t, fl.min(cap), lv, idx);
#                 if now == 0 {
#                     continue;
#                 }
# 
#                 self.g[v as usize][i as usize].cap -= now;
#                 self.g[dst as usize][opp as usize].cap += now;
#                 return now;
#             }
# 
#             0
#         }
# 
#         pub fn max_flow(&mut self, src: usize, dst: usize) -> u64 {
#             let mut flow = 0;
#             let mut aug;
#             let mut lv = vec![0; self.n];
#             let mut idx = vec![0; self.n];
# 
#             while self.bfs(src as u32, dst as u32, &mut lv) {
#                 idx.fill(0);
#                 loop {
#                     aug = self.dfs(src as u32, dst as u32, u64::MAX, &mut lv, &mut idx);
#                     if aug == 0 {
#                         break;
#                     }
#                     flow += aug;
#                 }
#             }
#             flow
#         }
#     }
# }
```

## Code
```rust,noplayground
mod dinic {
    //! Reference: https://github.com/justiceHui/SSU-SCCC-Study/blob/master/2022-winter-adv/slide/04.pdf

    use std::collections::VecDeque;

    #[derive(Clone)]
    pub struct Edge {
        pub dst: u32,
        pub opp: u32,
        pub cap: u64,
    }

    impl Edge {
        fn new(dst: usize, opp: usize, cap: u64) -> Self {
            Self {
                dst: dst as u32,
                opp: opp as u32,
                cap,
            }
        }
    }

    pub struct Dinic {
        pub n: usize,
        pub g: Vec<Vec<Edge>>,
    }

    impl Dinic {
        pub fn new(n: usize) -> Self {
            Self {
                n,
                g: vec![vec![]; n],
            }
        }

        pub fn add_edge(&mut self, s: usize, e: usize, cap: u64) {
            let sl = self.g[s].len();
            let el = self.g[e].len();
            self.g[s].push(Edge::new(e, el, cap));
            self.g[e].push(Edge::new(s, sl, 0));
        }

        fn bfs(&mut self, s: u32, t: u32, lv: &mut [u32]) -> bool {
            lv.fill(0);

            let mut queue = VecDeque::new();
            queue.push_back(s);
            lv[s as usize] = 1;

            while let Some(v) = queue.pop_front() {
                for e in self.g[v as usize].iter() {
                    if lv[e.dst as usize] == 0 && e.cap != 0 {
                        queue.push_back(e.dst);
                        lv[e.dst as usize] = lv[v as usize] + 1;
                    }
                }
            }

            lv[t as usize] != 0
        }

        fn dfs(&mut self, v: u32, t: u32, fl: u64, lv: &[u32], idx: &mut [u32]) -> u64 {
            if v == t || fl == 0 {
                return fl;
            }

            for i in idx[v as usize]..self.g[v as usize].len() as u32 {
                idx[v as usize] = i;

                let Edge { dst, opp, cap } = self.g[v as usize][i as usize];
                if lv[dst as usize] != lv[v as usize] + 1 || cap == 0 {
                    continue;
                }
                let now = self.dfs(dst, t, fl.min(cap), lv, idx);
                if now == 0 {
                    continue;
                }

                self.g[v as usize][i as usize].cap -= now;
                self.g[dst as usize][opp as usize].cap += now;
                return now;
            }

            0
        }

        pub fn max_flow(&mut self, src: usize, dst: usize) -> u64 {
            let mut flow = 0;
            let mut aug;
            let mut lv = vec![0; self.n];
            let mut idx = vec![0; self.n];

            while self.bfs(src as u32, dst as u32, &mut lv) {
                idx.fill(0);
                loop {
                    aug = self.dfs(src as u32, dst as u32, u64::MAX, &mut lv, &mut idx);
                    if aug == 0 {
                        break;
                    }
                    flow += aug;
                }
            }
            flow
        }
    }
}
```
