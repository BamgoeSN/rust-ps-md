# 2-SAT

Required snippets: [SCC](./scc.md#strongly-connected-components)
```rust,noplayground
struct TwoSat {
    n: usize,
    scc: SccGraph,
}

impl TwoSat {
    fn new(n: usize) -> Self {
        Self {
            n,
            scc: SccGraph::new(n << 1),
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
        self.scc.add_edge(
            (i << 1) + Self::judge(f, 0, 1),
            (j << 1) + Self::judge(g, 1, 0),
        );
        self.scc.add_edge(
            (j << 1) + Self::judge(g, 0, 1),
            (i << 1) + Self::judge(f, 1, 0),
        );
    }

    fn is_satisfiable(&self, answer: &mut [bool]) -> bool {
        let (_, ids) = self.scc.solve();
        for i in 0..self.n {
            if ids[i << 1] == ids[(i << 1) + 1] {
                return false;
            }
            answer[i] = ids[i << 1] < ids[(i << 1) + 1];
        }
        true
    }

    fn solve(&self) -> Option<Vec<bool>> {
        let mut answer = vec![false; self.n];
        let doable = self.is_satisfiable(&mut answer);
        if doable {
            Some(answer)
        } else {
            None
        }
    }
}
```