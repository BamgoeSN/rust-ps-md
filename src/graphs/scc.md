# Strongly Connected Components

```rust,noplayground
struct SccStack {
    stack: Vec<usize>,
    check: Vec<bool>,
}

impl SccStack {
    fn new(cap: usize) -> Self {
        Self {
            stack: vec![0; cap],
            check: vec![false; cap],
        }
    }

    fn len(&self) -> usize {
        self.stack.len()
    }

    fn push(&mut self, n: usize) {
        self.stack.push(n);
        self.check[n] = true;
    }

    fn get(&self) -> usize {
        self.stack[self.len() - 1]
    }

    fn pop(&mut self) -> usize {
        let tmp = self.stack.pop().unwrap();
        self.check[tmp] = false;
        tmp
    }

    fn is_in(&self, n: usize) -> bool {
        self.check[n]
    }
}

struct SccGraph {
    n: usize,
    graph: Vec<Vec<usize>>,
}

impl SccGraph {
    fn new(n: usize) -> Self {
        Self {
            n,
            graph: (0..n).map(|_| Vec::new()).collect(),
        }
    }

    fn add_edge(&mut self, from: usize, to: usize) {
        self.graph[from].push(to);
    }

    fn dfs(
        &self,
        curr: usize,
        gid: &mut usize,
        id: &mut Vec<usize>,
        low: &mut Vec<usize>,
        st: &mut SccStack,
        list: &mut Vec<Vec<usize>>,
    ) {
        st.push(curr);
        id[curr] = *gid;
        low[curr] = *gid;
        (*gid) += 1;

        for i in 0..self.graph[curr].len() {
            let next = self.graph[curr][i];
            if id[next] == self.n {
                self.dfs(next, gid, id, low, st, list);
            }
        }

        for i in 0..self.graph[curr].len() {
            let next = self.graph[curr][i];
            if st.is_in(next) {
                low[curr] = low[curr].min(low[next]);
            }
        }

        if id[curr] == low[curr] {
            let p = list.len();
            list.push(Vec::new());
            while st.len() != 0 && st.get() != curr {
                list[p].push(st.pop());
            }
            st.pop();
            list[p].push(curr);
        }
    }

    fn get_scc(&self) -> Vec<Vec<usize>> {
        let mut st = SccStack::new(self.n);
        let mut list: Vec<Vec<usize>> = Vec::new();
        let mut gid: usize = 0;
        let mut id = vec![self.n; self.n];
        let mut low = vec![usize::MAX; self.n];

        for x in 0..self.n {
            if id[x] != self.n {
                continue;
            }
            self.dfs(x, &mut gid, &mut id, &mut low, &mut st, &mut list)
        }

        list.reverse();
        list
    }

    /// SCC list and SCC IDs
    /// ids[node id] = id of SCC where the node is in
    fn solve(&self) -> (Vec<Vec<usize>>, Vec<usize>) {
        let list = self.get_scc();
        let mut ids = vec![0; self.n];
        for (i, l) in list.iter().enumerate() {
            for v in l.iter() {
                ids[*v] = i;
            }
        }
        (list, ids)
    }
}
```