# Segment Tree

Reference: AtCoder library <https://atcoder.github.io/ac-library/production/document_en/index.html>

A segment tree is a data structure for monoids \\( (S, \cdot : S \times S \rightarrow S, e \in S) \\). A monoid is an algebraic structure which follows the following conditions:
- \\(\cdot\\) is associative. That is, \\( (a \cdot b) \cdot c = a \cdot (b \cdot c) \\) for all \\( a, b, c \in S \\).
- There is the identity element \\(e\\) such that \\( a \cdot e = e \cdot a = a \\) for all \\( a \in S \\).
  
Given an array \\(A\\) of length \\(n\\) consists of the monoid \\(S\\) as described above, a segment tree on it can process the following queries in \\(O (\log{n})\\) time:
- Update an element
- Calculate the product of the elements of an interval

assuming that calculating the product of two elements takes \\(O(1)\\) time.

```rust,noplayground
use std::ops::Index;

fn ceil_pow2(n: usize) -> usize {
    let mut x: usize = 0;
    while (1 << x) < n {
        x += 1;
    }
    x
}

/// Represents each element of a segment tree, which should be a monoid $(S, \cdot)$.
pub trait Monoid {
    fn opr(&self, other: &Self) -> Self;
    fn e() -> Self;
}

/// Segment tree.
pub struct SegTree<S: Monoid> {
    n: usize,
    size: usize,
    log: usize,
    data: Vec<S>,
}

impl<S: Monoid + Clone> SegTree<S> {
    fn update(&mut self, k: usize) {
        self.data[k] = self.data[k << 1].opr(&self.data[(k << 1) + 1])
    }

    /// Initializes a segment tree from an array.
    pub fn new(arr: &Vec<S>) -> Self {
        let log = ceil_pow2(arr.len());
        let mut st: Self = SegTree {
            n: arr.len(),
            log,
            size: 1 << log,
            data: vec![S::e(); 1 << (log + 1)],
        };
        for (i, v) in arr.into_iter().enumerate() {
            st.data[st.size + i] = (*v).clone();
        }
        for i in (1..st.size).rev() {
            st.update(i);
        }
        st
    }

    /// Sets a value at index `i` to a value `v`.
    pub fn set(&mut self, i: usize, v: &S) {
        let i = i + self.size;
        self.data[i] = v.clone();
        for j in 1..=self.log {
            self.update(i >> j);
        }
    }

    /// Gets a reference of a value at index `i`.
    pub fn get(&self, i: usize) -> &S {
        &self[i]
    }

    /// Returns $A_l \cdot A_{l+1} \cdot \cdots \cdot A_{r-1}$. If $l=r$, then returns $e$.
    pub fn prod(&self, l: usize, r: usize) -> S {
        let (mut sml, mut smr) = (S::e(), S::e());
        let (mut l, mut r) = (l + self.size, r + self.size);

        while l < r {
            if l & 1 == 1 {
                sml = sml.opr(&self.data[l]);
                l += 1;
            }
            if r & 1 == 1 {
                r -= 1;
                smr = self.data[r].opr(&smr);
            }
            l >>= 1;
            r >>= 1;
        }
        sml.opr(&smr)
    }

    /// Returns $A_0 \cdot A_1 \cdot \cdots A_{n-1}$.
    pub fn all_prod(&self) -> S {
        self.data[1].clone()
    }

    pub fn max_right<C: Fn(&S) -> bool>(&self, l: usize, f: C) -> usize {
        if l == self.n {
            return self.n;
        }

        let mut l = l + self.size;
        let mut sm = S::e();

        loop {
            while l & 1 == 0 {
                l >>= 1;
            }
            if !f(&sm.opr(&self.data[l])) {
                while l < self.size {
                    l <<= 1;
                    if f(&sm.opr(&self.data[l])) {
                        sm = sm.opr(&self.data[l]);
                        l += 1;
                    }
                }
                return l - self.size;
            }
            sm = sm.opr(&self.data[l]);
            l += 1;
            if l & ((!l) + 1) == l {
                break;
            }
        }
        self.n
    }
    
    pub fn min_left<C: Fn(&S) -> bool>(&self, r: usize, f: C) -> usize {
        if r == 0 {
            return 0;
        }

        let mut r = r + self.size;
        let mut sm = S::e();

        loop {
            r -= 1;
            while r > 1 && r & 1 == 1 {
                r >>= 1;
            }
            if !f(&self.data[r].opr(&sm)) {
                while r < self.size {
                    r = (r << 1) + 1;
                    if f(&self.data[r].opr(&sm)) {
                        sm = self.data[r].opr(&sm);
                        r -= 1;
                    }
                }
                return r + 1 - self.size;
            }
            sm = self.data[r].opr(&sm);

            if r & ((!r) + 1) == r {
                break;
            }
        }
        0
    }
}

impl<T: Monoid + Clone> Index<usize> for SegTree<T> {
    type Output = T;

    fn index(&self, index: usize) -> &Self::Output {
        &self.data[index + self.size]
    }
}
```

## Using max_right and min_left
### max_right
Given an index \\(l\\) and a check function \\( f : S \rightarrow bool \\), `max_right` finds an index \\(r\\) such that satisfies both of the following conditions:
- \\(r=l\\) or \\( f \left( A_l \cdot A_{l+1} \cdots A_{r-1} \right) = true \\)
- \\(r=n\\) or \\( f \left( A_l \cdot A_{l+1} \cdots A_r \right) = false \\)

If \\(f\\) is monotone, this is the maximum \\(r\\) that satisfies \\( f \left( A_l \cdot A_{l+1} \cdots A_{r-1} \right) = true \\).

It should be guaranteed that \\(f(e)\\) is true, \\(0 \leq l \leq n\\), and \\(f\\) has no side effects i.e. calling \\(f\\) for the same value should always return the same result.

The search for \\(r\\) is done by binary search, so the time complexity of this function is \\( O(\log{n}) \\).

### min_left
Given an index \\(r\\) and a check function \\( f : S \rightarrow bool \\), `max_right` finds an index \\(r\\) such that satisfies both of the following conditions:
- \\(l=r\\) or \\( f \left( A_l \cdot A_{l+1} \cdots A_{r-1} \right) = true \\)
- \\(l=0\\) or \\( f \left( A_{l-1} \cdot A_l \cdots A_{r-1} \right) = false \\)

If \\(f\\) is monotone, this is the minimum \\(l\\) that satisfies \\( f \left( A_l \cdot A_{l+1} \cdots A_{r-1} \right) = true \\).

It should be guaranteed that \\(f(e)\\) is true, \\(0 \leq r \leq n\\), and \\(f\\) has no side effects i.e. calling \\(f\\) for the same value should always return the same result.

The search for \\(l\\) is done by binary search, so the time complexity of this function is \\( O(\log{n}) \\).