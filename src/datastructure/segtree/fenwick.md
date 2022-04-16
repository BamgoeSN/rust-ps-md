# Fenwick Tree

Given an integer array \\(A\\) of length \\(n\\), a Fenwick tree processes the following queries in \\(O(\log{n})\\) time:
- Add a certain amount to an element
- Calculate the sum of the elements of an interval

A Fenwick tree uses half the memory of a segment tree, but the performance in terms of time is just about the same.

Required snippets: [Zero Trait](../../misc/zero_one_trait.md#zero)
```rust,noplayground
struct Fenwick<T: Zero + AddAssign + Sub<Output = T>> {
    n: usize,
    data: Vec<T>,
}

impl<T: Zero + AddAssign + Sub<Output = T>> Fenwick<T> {
    fn new(n: usize) -> Self {
        Self {
            n,
            data: vec![T::zero(); n],
        }
    }

    fn add(&mut self, idx: usize, val: T) {
        let mut idx = idx + 1;
        while idx <= self.n {
            self.data[idx - 1] += val;
            idx += idx & (!idx + 1);
        }
    }

    fn sum(&self, l: usize, r: usize) -> T {
        self.inner_sum(r) - self.inner_sum(l)
    }

    fn inner_sum(&self, mut r: usize) -> T {
        let mut s: T = T::zero();
        while r > 0 {
            s += self.data[r - 1];
            r -= r & (!r + 1);
        }
        s
    }
}
```