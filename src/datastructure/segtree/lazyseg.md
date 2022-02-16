# Lazy Segment Tree

Reference: AtCoder library <https://atcoder.github.io/ac-library/production/document_en/index.html>

A lazy segment tree is a data struture for a pair of a monoid \\( (S, \cdot : S \times S \rightarrow S, e \in S) \\) and a set \\(F\\) of \\(S \rightarrow S\\) mappings that satisfies the following properties:
- \\(F\\) contains the identity mapping \\(Id\\) such that \\( Id(x) = x \\) for all \\(x\in S\\).
- \\(F\\) is closed under composition. That is, \\( f \circ g \in F \\) for all \\( f, g \in F \\).
- \\( f (x \cdot y) = f(x) \cdot f(y) \\) hold for all \\(f \in F \\) and \\( x, y \in S \\).

Given an array \\(A\\) of length \\(n\\) consists of the monoid \\(S\\) as described above, a segment tree on it can process the following queries in \\(O (\log{n})\\) time:
- Apply the mapping \\( f \in F \\) on all the elements of an interval
- Calculate the product of the elements of an interval

assuming that calculating the product of two elements takes \\(O(1)\\) time.

## Snippet
```rust,noplayground
fn ceil_pow2(n: usize) -> usize {
    let mut x: usize = 0;
    while (1 << x) < n {
        x += 1;
    }
    x
}

/// Represents a monoid in a lazy segment tree $(S, \cdot)$.
pub trait Monoid {
    fn opr(&self, other: &Self) -> Self;
    fn e() -> Self;
}

/// Represents a map $F$ acting on the monoid $(S, \cdot)$.
pub trait Mapping<T: Monoid> {
    fn map(&self, apply: &T) -> T;
    fn compos(&self, other: &Self) -> Self;
    fn id() -> Self;
}

/// Lazy segment tree for the monoid $(S, \cdot)$ and the map $F$.
pub struct LazySeg<S: Monoid, F: Mapping<S>> {
    n: usize,
    size: usize,
    log: usize,
    data: Vec<S>,
    lazy: Vec<F>,
}

impl<S, F> LazySeg<S, F>
where
    S: Monoid + Clone,
    F: Mapping<S> + Clone,
{
    fn update(&mut self, k: usize) {
        self.data[k] = self.data[k << 1].opr(&self.data[(k << 1) + 1]);
    }

    fn all_apply(&mut self, k: usize, f: &F) {
        self.data[k] = f.map(&self.data[k]);
        if k < self.size {
            self.lazy[k] = f.compos(&self.lazy[k]);
        }
    }

    fn push(&mut self, k: usize) {
        self.all_apply(k << 1, &self.lazy[k].clone());
        self.all_apply((k << 1) + 1, &self.lazy[k].clone());
        self.lazy[k] = F::id();
    }

    /// Initializes the lazy segment tree from the given array.
    pub fn new(arr: &Vec<S>) -> Self {
        let log = ceil_pow2(arr.len());
        let mut ls: Self = LazySeg {
            n: arr.len(),
            log,
            size: 1 << log,
            data: vec![S::e(); 1 << (log + 1)],
            lazy: vec![F::id(); 1 << log],
        };
        for (i, v) in arr.into_iter().enumerate() {
            ls.data[ls.size + i] = (*v).clone();
        }
        for i in (1..ls.size).rev() {
            ls.update(i);
        }
        ls
    }

    /// Sets a value at the index `i` to `v`.
    pub fn set(&mut self, i: usize, v: &S) {
        let i = i + self.size;
        for j in (1..=self.log).rev() {
            self.push(i >> j);
        }
        self.data[i] = v.clone();
        for j in 1..=self.log {
            self.update(i >> j);
        }
    }

    /// Returns a reference to the value at the index `i`.
    pub fn get(&mut self, i: usize) -> &S {
        let i = i + self.size;
        for j in (1..=self.log).rev() {
            self.push(i >> j);
        }
        &self.data[i]
    }

    /// Returns a product of elements in [l, r).
    pub fn prod(&mut self, l: usize, r: usize) -> S {
        if l == r {
            return S::e();
        }

        let (mut l, mut r) = (l + self.size, r + self.size);

        for i in (1..=self.log).rev() {
            if ((l >> i) << i) != l {
                self.push(l >> i);
            }
            if ((r >> i) << i) != r {
                self.push((r - 1) >> i);
            }
        }

        let (mut sml, mut smr) = (S::e(), S::e());
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

    /// Returns the product of every elements.
    pub fn all_prod(&self) -> S {
        self.data[1].clone()
    }

    /// Apply a map `f` to the element at `i`.
    pub fn apply(&mut self, i: usize, f: &F) {
        let i = i + self.size;
        for j in (1..=self.log).rev() {
            self.push(i >> j);
        }
        self.data[i] = f.map(&self.data[i]);
        for j in 1..=self.log {
            self.update(i >> j);
        }
    }

    /// Apply a map `f` to the element in an interval [l, r).
    pub fn apply_range(&mut self, l: usize, r: usize, f: &F) {
        if l == r {
            return;
        }

        let (mut l, mut r) = (l + self.size, r + self.size);

        for i in (1..=self.log).rev() {
            if ((l >> i) << i) != l {
                self.push(l >> i);
            }
            if ((r >> i) << i) != r {
                self.push((r - 1) >> i);
            }
        }

        let (l2, r2) = (l, r);
        while l < r {
            if l & 1 == 1 {
                self.all_apply(l, f);
                l += 1;
            }
            if r & 1 == 1 {
                r -= 1;
                self.all_apply(r, f);
            }
            l >>= 1;
            r >>= 1;
        }
        l = l2;
        r = r2;

        for i in 1..=self.log {
            if ((l >> i) << i) != l {
                self.update(l >> i);
            }
            if ((r >> i) << i) != r {
                self.update((r - 1) >> i);
            }
        }
    }
    
    pub fn max_right<C: Fn(&S) -> bool>(&mut self, l: usize, g: C) -> usize {
        if l == self.n {
            return self.n;
        }

        let mut l = l + self.size;
        for i in (1..=self.log).rev() {
            self.push(l >> i);
        }

        let mut sm = S::e();
        loop {
            while l & 1 == 0 {
                l >>= 1;
            }
            if !g(&sm.opr(&self.data[l])) {
                while l < self.size {
                    self.push(l);
                    l <<= 1;
                    if g(&sm.opr(&self.data[l])) {
                        sm = sm.opr(&self.data[l]);
                        l += 1;
                    }
                }
                return l - self.size;
            }
            sm = sm.opr(&self.data[l]);
            l += 1;

            if l & ((!l) + 1) != l {
                break;
            }
        }
        self.n
    }
    
    pub fn min_left<C: Fn(&S) -> bool>(&mut self, r: usize, g: C) -> usize {
        if r == 0 {
            return 0;
        }

        let mut r = r + self.size;
        for i in (1..=self.log).rev() {
            self.push((r - 1) >> i);
        }

        let mut sm = S::e();
        loop {
            r -= 1;
            while r > 1 && r & 1 == 1 {
                r >>= 1;
            }
            if !g(&self.data[r].opr(&sm)) {
                while r < self.size {
                    self.push(r);
                    r = (r << 1) + 1;
                    if g(&self.data[r].opr(&sm)) {
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

## Examples of monoid and mapping implementation
### \\(ax+b\\) lazy segment tree
```rust,noplayground
#[derive(Clone)]
struct Elm {
    len: i128,
    sum: i128,
}

impl Monoid for Elm {
    fn opr(&self, other: &Self) -> Self {
        Self {
            len: self.len + other.len,
            sum: self.sum + other.sum,
        }
    }
    fn e() -> Self {
        Self { len: 0, sum: 0 }
    }
}

#[derive(Clone)]
struct Map {
    mul: i128,
    add: i128,
}

impl Mapping<Elm> for Map {
    fn map(&self, apply: &Elm) -> Elm {
        Elm {
            len: apply.len,
            sum: self.mul * apply.sum + self.add * apply.len,
        }
    }
    fn compos(&self, other: &Self) -> Self {
        Self {
            mul: self.mul * other.mul,
            add: self.mul * other.add + self.add,
        }
    }
    fn id() -> Self {
        Self { mul: 1, add: 0 }
    }
}
```