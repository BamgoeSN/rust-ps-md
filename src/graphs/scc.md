# Strongly Connected Components

```rust,noplayground
#[derive(Debug)]
struct Graph<T> {
    n: u32,
    first: Vec<u32>,
    edge: Vec<(u32, u32, T)>, // (to, prev, data)
}

impl<T> Graph<T> {
    fn new(n: u32, e: u32) -> Self {
        Self {
            n,
            first: vec![u32::MAX; n as usize],
            edge: Vec::with_capacity(e as usize),
        }
    }

    fn add_edge(&mut self, from: u32, to: u32, data: T) {
        let prev = std::mem::replace(&mut self.first[from as usize], self.edge.len() as u32);
        self.edge.push((to, prev, data));
    }

    fn neighbor(&self, of: u32) -> Neighbor<T> {
        Neighbor {
            graph: self,
            next_edge: self.first[of as usize],
        }
    }
}

struct Neighbor<'g, T> {
    graph: &'g Graph<T>,
    next_edge: u32,
}

impl<'g, T> Iterator for Neighbor<'g, T> {
    type Item = (u32, &'g T);

    fn next(&mut self) -> Option<Self::Item> {
        let (to, next_edge, data) = self.graph.edge.get(self.next_edge as usize)?;
        self.next_edge = *next_edge;
        Some((*to, data))
    }
}

struct SccStack {
    stack: Vec<u32>,
    check: Vec<bool>,
}

impl SccStack {
    fn new(cap: u32) -> Self {
        Self {
            stack: vec![0; cap as usize],
            check: vec![false; cap as usize],
        }
    }

    fn len(&self) -> u32 {
        self.stack.len() as u32
    }

    fn push(&mut self, n: u32) {
        self.stack.push(n);
        self.check[n as usize] = true;
    }

    fn get(&self) -> u32 {
        self.stack[self.len() as usize - 1]
    }

    fn pop(&mut self) -> u32 {
        let tmp = self.stack.pop().unwrap();
        self.check[tmp as usize] = false;
        tmp
    }

    fn is_in(&self, n: u32) -> bool {
        self.check[n as usize]
    }
}

struct SCC<'g, T> {
    graph: &'g Graph<T>,
    scc_list: Vec<Vec<u32>>,
    scc_ids: Vec<u32>,
}

impl<'g, T: 'g> SCC<'g, T> {
    fn new(graph: &'g Graph<T>) -> Self {
        let mut st = SccStack::new(graph.n);
        let mut list: Vec<Vec<u32>> = Vec::new();
        let mut gid: u32 = 0;
        let mut id = vec![graph.n; graph.n as usize];
        let mut low = vec![u32::MAX; graph.n as usize];

        for x in 0..graph.n {
            if id[x as usize] != graph.n {
                continue;
            }
            Self::dfs(graph, x, &mut gid, &mut id, &mut low, &mut st, &mut list)
        }
        list.reverse();

        let mut ids = vec![0u32; graph.n as usize];
        for (i, l) in list.iter().enumerate() {
            for &v in l.iter() {
                ids[v as usize] = i as u32;
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
        list: &mut Vec<Vec<u32>>,
    ) {
        st.push(curr);
        id[curr as usize] = *gid;
        low[curr as usize] = *gid;
        (*gid) += 1;

        for (next, _) in graph.neighbor(curr) {
            if id[next as usize] == graph.n {
                Self::dfs(graph, next, gid, id, low, st, list);
            }
        }

        for (next, _) in graph.neighbor(curr) {
            if st.is_in(next) {
                low[curr as usize] = low[curr as usize].min(low[next as usize]);
            }
        }

        if id[curr as usize] == low[curr as usize] {
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