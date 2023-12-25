# Heavy-Light Decomposition

## Code
```rust,noplayground
/// Reference: https://codeforces.com/blog/entry/53170
mod hld {
	pub struct Hld {
		pub root: usize,
		/// `size[i]`: The size of a subtree rooted with `i`.
		pub size: Vec<usize>,
		/// `dep[i]`: A depth of `i`.
		pub dep: Vec<usize>,
		/// `par[i]`: A parent node of `i`.
		pub par: Vec<usize>,
		/// `top[i]`: The highest node of a chain `i` is in.
		pub top: Vec<usize>,
		/// `ein[i]`: DFS ordering of `i`.
		pub ein: Vec<usize>,
		/// `eout[i]`: The order when DFS exited from `i`.
		pub eout: Vec<usize>,
		/// If `rin[v] == i`, then `ein[i] == v`.
		pub rin: Vec<usize>,
	}

	impl Hld {
		pub fn new(root: usize, graph: &mut [Vec<usize>]) -> Self {
			fn dfs1(u: usize, p: usize, graph: &mut [Vec<usize>], h: &mut Hld) {
				h.size[u] = 1;
				h.par[u] = p;
				for i in 0..graph[u].len() {
					let v = graph[u][i];
					if v != p {
						h.dep[v] = h.dep[u] + 1;
						dfs1(v, u, graph, h);
						h.size[u] += h.size[v];
						if h.size[v] > h.size[graph[u][0]] {
							graph[u].swap(i, 0);
						}
					}
				}
			}

			fn dfs2(u: usize, p: usize, cnt: &mut usize, graph: &[Vec<usize>], h: &mut Hld) {
				h.ein[u] = *cnt;
				h.rin[*cnt] = u;
				*cnt += 1;
				for &v in graph[u].iter().filter(|&&v| v != p) {
					h.top[v] = if v == graph[u][0] { h.top[u] } else { v };
					dfs2(v, u, cnt, graph, h);
				}
				h.eout[u] = *cnt;
			}

			let n = graph.len();
			let [size, dep, par, top, ein, eout, rin] = [0; 7].map(|_| vec![0; n]);
			let mut ret = Hld { root, size, dep, par, top, ein, eout, rin };

			dfs1(root, n, graph, &mut ret);
			dfs2(root, n, &mut 0, graph, &mut ret);
			ret
		}

		/// Returns ranges which constitute a path between `u` and `v`.
		/// Every range is a half-open intervals, i.e. `[l, r)`, and indices of the ranges are matched with `self.ein`.
		/// The ranges are for path queries on "vertices". For "edges", the last range should be truncated at left by 1.
		pub fn chains(&self, u: usize, v: usize) -> impl Iterator<Item = (usize, usize)> + '_ {
			let (mut a, mut b, mut k) = (u, v, true);
			std::iter::from_fn(move || {
				k.then(|| {
					if self.top[a] != self.top[b] {
						if self.dep[self.top[a]] < self.dep[self.top[b]] {
							std::mem::swap(&mut a, &mut b);
						}
						let st = self.top[a];
						let ret = (self.ein[st], self.ein[a] + 1);
						a = self.par[st];
						ret
					} else {
						if self.dep[a] > self.dep[b] {
							std::mem::swap(&mut a, &mut b);
						}
						k = false;
						(self.ein[a], self.ein[b] + 1)
					}
				})
			})
		}

		pub fn lca(&self, u: usize, v: usize) -> usize { self.chains(u, v).last().map(|(a, _)| self.rin[a]).unwrap() }

		pub fn lca_alt_root(&self, r: usize, u: usize, v: usize) -> usize {
			if r == self.root {
				return self.lca(u, v);
			}
			let (uv, ru, rv) = (self.lca(u, v), self.lca(r, u), self.lca(r, v));
			if rv == uv {
				ru
			} else if ru == uv {
				rv
			} else {
				uv
			}
		}
	}
}
```