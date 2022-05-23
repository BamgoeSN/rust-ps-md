# 2-SAT

Required snippets: [SCC](./scc.md#strongly-connected-components)
```rust,noplayground
struct TwoSat {
    n: u32,
    graph: Graph<()>,
}

impl TwoSat {
    fn new(n: u32, clause_num: u32) -> Self {
        Self {
            n,
            graph: Graph::new(n << 1, clause_num << 1),
        }
    }

    fn judge(f: bool, a: u32, b: u32) -> u32 {
        if f {
            a
        } else {
            b
        }
    }

    fn add_clause(&mut self, i: u32, f: bool, j: u32, g: bool) {
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

    fn solve(self) -> Option<Vec<bool>> {
        let mut answer = vec![false; self.n as usize];

        let scc = SCC::new(&self.graph);
        let ids = &scc.scc_ids;

        for i in 0..self.n {
            if ids[(i as usize) << 1] == ids[((i as usize) << 1) + 1] {
                return None;
            }
            answer[i as usize] = ids[(i as usize) << 1] < ids[((i as usize) << 1) + 1];
        }
        Some(answer)
    }
}
```