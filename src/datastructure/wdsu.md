# Weighted DSU

Disjoint set union where vertices have their own "potential value". The potential value of a vertex \\(i\\) is denoted as \\(w(i)\\), and the differences of potential values between vertices in a same group are always well defined.

`fn union(&mut self, u: usize, v: usize, dw: i64)` addes a "rule" stating that \\(u\\) and \\(v\\) are in a same group, and \\(w(u) - w(v) = dw\\). Based on posed rules ahead, `fn get_pot_diff(&mut self, u: usize, v: usize) -> Option<i64>` returns \\(w(u) - w(v)\\) if \\(u\\) and \\(v\\) are in a same group.

`potdiff[i]` in the code is defined as \\(w(i) - w(r_i)\\), where \\(r_i\\) is a root of a group \\(i\\) is in.

## Code
```rust,noplayground
struct WeightDSU {
	parent: Vec<usize>,
	size: Vec<usize>,
	num: usize,
	potdiff: Vec<i64>, // potdiff[x] == w(x) - w(p)
}

impl WeightDSU {
	fn new(n: usize) -> Self {
		Self {
			parent: (0..n).collect(),
			size: vec![1; n],
			num: n,
			potdiff: vec![0; n],
		}
	}

	// Returns the root of x.
	fn find_root(&mut self, mut x: usize) -> usize {
		while self.parent[x] != x {
			let p = self.parent[x];
			self.potdiff[x] += self.potdiff[p];
			self.parent[x] = self.parent[p];
			x = self.parent[x];
		}
		x
	}

	// Returns the root of x, namely xr, with w(x) - w(xr).
	fn find_root_with_pdiff(&mut self, mut x: usize) -> (usize, i64) {
		let mut pd = 0;
		while self.parent[x] != x {
			let p = self.parent[x];
			self.potdiff[x] += self.potdiff[p];
			self.parent[x] = self.parent[p];
			pd += self.potdiff[x];
			x = self.parent[x];
		}
		(x, pd)
	}

	// Unions groups of u and v, with w(u) - w(v) = dw. If u and v are already in a same group,
	// if w(u) - w(v) == dv then it returns Some(true), otherwise Some(false). If they weren't
	// in a same group, then it returns None and unions two groups following the given dw.
	fn union(&mut self, u: usize, v: usize, dw: i64) -> Option<bool> {
		let (ur, pu) = self.find_root_with_pdiff(u);
		let (vr, pv) = self.find_root_with_pdiff(v);
		let nw = dw - pu + pv;
		if ur != vr {
			self.num -= 1;
			if self.size[ur] < self.size[vr] {
				self.parent[ur] = vr;
				self.size[vr] += self.size[ur];
				self.potdiff[ur] = nw;
			} else {
				self.parent[vr] = ur;
				self.size[ur] += self.size[vr];
				self.potdiff[vr] = -nw;
			}
			None
		} else {
			Some(nw == 0)
		}
	}

	// Returns the size of a group x is in.
	fn get_size(&mut self, x: usize) -> usize {
		let r = self.find_root(x);
		self.size[r]
	}

	// Returns Some(w(u) - w(v)) if u and v are in the same group, otherwise None.
	fn get_pot_diff(&mut self, u: usize, v: usize) -> Option<i64> {
		let (ur, pu) = self.find_root_with_pdiff(u);
		let (vr, pv) = self.find_root_with_pdiff(v);
		if ur == vr {
			Some(pu - pv)
		} else {
			None
		}
	}
}
```
