# Strongly Connected Components

```rust,noplayground
#[derive(Debug)]
struct Graph<T> {
    n: usize,
    first: Vec<usize>,
    edge: Vec<(usize, usize, T)>, // (to, prev, data)
}

impl<T> Graph<T> {
    fn new(n: usize, e: usize) -> Self {
        Self {
            n,
            first: vec![usize::MAX; n],
            edge: Vec::with_capacity(e),
        }
    }

    fn add_edge(&mut self, from: usize, to: usize, data: T) {
        let prev = std::mem::replace(&mut self.first[from], self.edge.len());
        self.edge.push((to, prev, data));
    }

    fn neighbor(&self, of: usize) -> Neighbor<T> {
        Neighbor {
            graph: self,
            next_edge: self.first[of],
        }
    }
}

struct Neighbor<'g, T> {
    graph: &'g Graph<T>,
    next_edge: usize,
}

impl<'g, T> Iterator for Neighbor<'g, T> {
    type Item = (usize, &'g T);

    fn next(&mut self) -> Option<Self::Item> {
        let (to, next_edge, data) = self.graph.edge.get(self.next_edge)?;
        self.next_edge = *next_edge;
        Some((*to, data))
    }
}

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

struct SCC<'g, T> {
    graph: &'g Graph<T>,
    scc_list: Vec<Vec<usize>>,
    scc_ids: Vec<usize>,
}

impl<'g, T: 'g> SCC<'g, T> {
    fn new(graph: &'g Graph<T>) -> Self {
        let mut st = SccStack::new(graph.n);
        let mut list: Vec<Vec<usize>> = Vec::new();
        let mut gid: usize = 0;
        let mut id = vec![graph.n; graph.n];
        let mut low = vec![usize::MAX; graph.n];

        for x in 0..graph.n {
            if id[x] != graph.n {
                continue;
            }
            Self::dfs(graph, x, &mut gid, &mut id, &mut low, &mut st, &mut list)
        }
        list.reverse();

        let mut ids = vec![0; graph.n];
        for (i, l) in list.iter().enumerate() {
            for v in l.iter() {
                ids[*v] = i;
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

        for (next, _) in graph.neighbor(curr) {
            if id[next] == graph.n {
                Self::dfs(graph, next, gid, id, low, st, list);
            }
        }

        for (next, _) in graph.neighbor(curr) {
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
}
```