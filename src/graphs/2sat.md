# 2-SAT

Required snippets: [SCC](./scc.md#strongly-connected-components)
```rust,noplayground
struct TwoSat {
    n: usize,
    graph: Graph<()>,
}

impl TwoSat {
    fn new(n: usize, clause_num: usize) -> Self {
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

    fn add_clause(&mut self, i: usize, f: bool, j: usize, g: bool) {
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
        let mut answer = vec![false; self.n];

        let scc = SCC::new(&self.graph);
        let ids = &scc.scc_ids;

        for i in 0..self.n {
            if ids[i << 1] == ids[(i << 1) + 1] {
                return None;
            }
            answer[i] = ids[i << 1] < ids[(i << 1) + 1];
        }
        Some(answer)
    }
}
```