# Closest Points

## Code
```rust,noplayground
/// Returns the two farthest points from `p` by indices, or `None` if `p.len() <= 1`.
fn closest_pair(p: &[P]) -> Option<[usize; 2]> {
	let mut u: Vec<usize> = (0..p.len()).collect();
	u.sort_unstable_by_key(|&i| p[i]);
	closest_pair_recur(p, &mut u).map(|x| x.1)
}

/// Sorts `p` by y coord of each point, and returns the closest pair and their distance squared.
fn closest_pair_recur(p: &[P], u: &mut [usize]) -> Option<(I, [usize; 2])> {
	if u.len() <= 1 {
		return None;
	}
	let dist = |a: P, b: P| dot(sub(a, b), sub(a, b));

	// Divide
	let m = u.len() / 2;
	let pivot = p[u[m]][0];
	let (l, r) = u.split_at_mut(m);
	let [xl, xr] = [l, r].map(|x| closest_pair_recur(p, x));
	let mut d = [xl, xr].into_iter().flatten().min_by_key(|x| x.0);
	// Now l and r are sorted by y coords

	// Merge two lists into one while keeping the y coords sorted
	let (l, r) = u.split_at(m);
	let [mut lit, mut rit] = [l, r].map(|x| x.to_vec().into_iter().peekable());
	let mut x = u.iter_mut();
	while let (Some(&l), Some(&r)) = (lit.peek(), rit.peek()) {
		*(x.next().unwrap()) = if p[l][1] <= p[r][1] {
			lit.next();
			l
		} else {
			rit.next();
			r
		};
	}
	lit.for_each(|l| *(x.next().unwrap()) = l);
	rit.for_each(|r| *(x.next().unwrap()) = r);
	// Now p is sorted by y coords

	// Conquer
	let mut hist: [Option<usize>; 3] = [None; 3];
	let mut ptr = (0..3).cycle();
	for &i in u.iter() {
		if d.map_or(true, |(d, _)| (p[i][0] - pivot).pow(2) < d) {
			let j = hist.iter().flatten().min_by_key(|&&j| dist(p[i], p[j]));
			if let Some(&j) = j {
				let nd = dist(p[i], p[j]);
				if d.map_or(true, |(d, _)| nd < d) {
					d = Some((nd, [i, j]));
				}
			}
			hist[ptr.next().unwrap()] = Some(i);
		}
	}

	d
}
```

---

Last modified on 231225.