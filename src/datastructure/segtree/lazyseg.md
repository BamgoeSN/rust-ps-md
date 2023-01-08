# Lazy Segment Tree

Reference: AtCoder library <https://atcoder.github.io/ac-library/production/document_en/index.html>

A lazy segment tree is a data struture for a pair of [a monoid](./segtree.md) \\( (S, \cdot : S \times S \rightarrow S, e \in S) \\) and a set \\(F\\) of \\(S \rightarrow S\\) mappings that satisfies the following properties:
- \\(F\\) contains the identity mapping \\(Id\\) such that \\( Id(x) = x \\) for all \\(x\in S\\).
- \\(F\\) is closed under composition. That is, \\( f \circ g \in F \\) for all \\( f, g \in F \\).
- \\( f (x \cdot y) = f(x) \cdot f(y) \\) hold for all \\(f \in F \\) and \\( x, y \in S \\).

Given an array \\(A\\) of length \\(n\\) consists of the monoid \\(S\\) as described above, a segment tree on it can process the following queries in \\(O (\log{n})\\) time:
- Apply the mapping \\( f \in F \\) on all the elements of an interval
- Calculate the product of the elements of an interval

assuming that calculating the product of two elements takes \\(O(1)\\) time.

## Example

```rust
# use lazyseg::*;
# 
// ax+b lazy segment tree
type P = (i64, i64);

// P = (sum, len)
impl LzMonoid for P {
    fn e() -> Self { (0, 0) }
    fn opr_lhs(&mut self, rhs: &Self) {
        self.0 += rhs.0; self.1 += rhs.1;
    }
    fn opr_rhs(&mut self, lhs: &Self) {
        self.0 += lhs.0; self.1 += lhs.1;
    }
}

// P = (a, b) equiv. to applying ax+b on LzMonoids
impl LzMap<P> for P {
    fn id() -> Self { (1, 0) }
    fn map(&self, apply: &mut P) {
        apply.0 = self.0 * apply.0 + self.1 * apply.1;
    }
    fn compos_lhs(&mut self, rhs: &Self) {
        let (a, b, c, d) = (self.0, self.1, rhs.0, rhs.1);
        *self = (a * c, a * d + b);
    }
    fn compos_rhs(&mut self, lhs: &Self) {
        let (a, b, c, d) = (lhs.0, lhs.1, self.0, self.1);
        *self = (a * c, a * d + b);
    }
}

# fn main() {
let mut st: LazySeg<_, P> = LazySeg::new((0..10).map(|v| (v, 1)).collect());
println!("{}", st.get(3).0); // 3
println!("{}", st.prod(2..6).0); // 14

// Add 5 to 1..6
st.apply_range(1..6, &(1, 5));
// Multiply 6.. by 4
st.apply_range(6.., &(4, 0));
// 0 6 7 8 9 10 24 28 32 36

let r = st.max_right(4, |&(v, _)| v < 43);
println!("{}", r); // 6
let l = st.min_left(6, |&(v, _)| v < 32);
println!("{}", l); // 3
# }
# 
# mod lazyseg {
#     use std::ops::RangeBounds;
# 
#     fn ceil_pow2(n: usize) -> u32 {
#         let mut x = 0;
#         while 1 << x < n {
#             x += 1;
#         }
#         x
#     }
# 
#     fn range_to_bounds(n: usize, range: impl RangeBounds<usize>) -> (usize, usize) {
#         use std::ops::Bound::*;
# 
#         let l = match range.start_bound() {
#             Included(&v) => v,
#             Excluded(&v) => v + 1,
#             Unbounded => 0,
#         };
# 
#         let r = match range.end_bound() {
#             Included(&v) => v + 1,
#             Excluded(&v) => v,
#             Unbounded => n,
#         };
# 
#         (l, r)
#     }
# 
#     pub trait LzMonoid: Sized {
#         fn e() -> Self;
#         fn opr_lhs(&mut self, rhs: &Self);
#         fn opr_rhs(&mut self, lhs: &Self);
#         fn opr(lhs: &Self, rhs: &Self) -> Self {
#             let mut ret = Self::e();
#             ret.opr_rhs(lhs);
#             ret.opr_lhs(rhs);
#             ret
#         }
#         fn opr_set(&mut self, lhs: &Self, rhs: &Self) {
#             *self = Self::opr(lhs, rhs);
#         }
#     }
# 
#     pub trait LzMap<S>: Sized {
#         fn id() -> Self;
#         fn map(&self, apply: &mut S);
#         fn compos_lhs(&mut self, rhs: &Self);
#         fn compos_rhs(&mut self, lhs: &Self);
#         fn compos(lhs: &Self, rhs: &Self) -> Self {
#             let mut ret = Self::id();
#             ret.compos_rhs(lhs);
#             ret.compos_lhs(rhs);
#             ret
#         }
#         fn opr_self(&mut self, lhs: &Self, rhs: &Self) {
#             *self = Self::compos(lhs, rhs);
#         }
#     }
# 
#     pub struct LazySeg<S, F> {
#         n: usize,
#         size: usize,
#         log: u32,
#         data: Vec<S>,
#         lazy: Vec<F>,
#     }
# 
#     impl<S, F> LazySeg<S, F>
#     where
#         S: LzMonoid,
#         F: LzMap<S>,
#     {
#         fn update(&mut self, k: usize) {
#             let val = S::opr(&self.data[k << 1], &self.data[(k << 1) + 1]);
#             self.data[k] = val;
#         }
# 
#         fn all_apply(&mut self, k: usize, f: &F) {
#             f.map(&mut self.data[k]);
#             if k < self.size {
#                 self.lazy[k].compos_rhs(f);
#             }
#         }
# 
#         fn all_apply_unsafe(size: usize, k: usize, f: usize, data: &mut [S], lazy: &mut [F]) {
#             // f < k
#             lazy[f].map(&mut data[k]);
#             if k < size {
#                 let (fs, ks) = lazy.split_at_mut(k);
#                 ks[0].compos_rhs(&fs[f]);
#             }
#         }
# 
#         fn push(&mut self, k: usize) {
#             Self::all_apply_unsafe(self.size, k << 1, k, &mut self.data, &mut self.lazy);
#             Self::all_apply_unsafe(self.size, (k << 1) + 1, k, &mut self.data, &mut self.lazy);
#             self.lazy[k] = F::id();
#         }
# 
#         pub fn new(arr: Vec<S>) -> Self {
#             let n = arr.len();
#             let log = ceil_pow2(n);
#             let size = 1 << log;
#             let stsize = 1 << (log + 1);
# 
#             let mut data = Vec::with_capacity(stsize);
#             data.extend((0..size).map(|_| S::e()));
#             data.extend(arr.into_iter());
#             data.extend((data.len()..stsize).map(|_| S::e()));
# 
#             let mut ls = Self {
#                 n,
#                 log,
#                 size,
#                 data,
#                 lazy: Vec::from_iter((0..size).map(|_| F::id())),
#             };
#             for i in (1..size).rev() {
#                 ls.update(i);
#             }
#             ls
#         }
# 
#         pub fn set(&mut self, i: usize, v: S) {
#             let i = i + self.size;
#             for j in (1..=self.log).rev() {
#                 self.push(i >> j);
#             }
#             self.data[i] = v;
#             for j in 1..=self.log {
#                 self.update(i >> j);
#             }
#         }
# 
#         pub fn get(&mut self, i: usize) -> &S {
#             let i = i + self.size;
#             for j in (1..=self.log).rev() {
#                 self.push(i >> j);
#             }
#             &self.data[i]
#         }
# 
#         pub fn prod(&mut self, range: impl RangeBounds<usize>) -> S {
#             let (l, r) = range_to_bounds(self.size, range);
# 
#             if l == 0 && r == self.size {
#                 let mut ret = S::e();
#                 ret.opr_lhs(&self.data[1]);
#                 return ret;
#             } else if l == r {
#                 return S::e();
#             }
# 
#             let (mut l, mut r) = (l + self.size, r + self.size);
# 
#             for i in (1..=self.log).rev() {
#                 if ((l >> i) << i) != l {
#                     self.push(l >> i);
#                 }
#                 if ((r >> i) << i) != r {
#                     self.push((r - 1) >> i);
#                 }
#             }
# 
#             let (mut sml, mut smr) = (S::e(), S::e());
#             while l < r {
#                 if l & 1 == 1 {
#                     sml.opr_lhs(&self.data[l]);
#                     l += 1;
#                 }
#                 if r & 1 == 1 {
#                     r -= 1;
#                     smr.opr_rhs(&self.data[r]);
#                 }
#                 l >>= 1;
#                 r >>= 1;
#             }
# 
#             S::opr(&sml, &smr)
#         }
# 
#         pub fn apply(&mut self, i: usize, f: &F) {
#             let i = i + self.size;
#             for j in (1..=self.log).rev() {
#                 self.push(i >> j);
#             }
#             f.map(&mut self.data[i]);
#             for j in 1..=self.log {
#                 self.update(i >> j);
#             }
#         }
# 
#         pub fn apply_range(&mut self, range: impl RangeBounds<usize>, f: &F) {
#             let (l, r) = range_to_bounds(self.size, range);
# 
#             if l == r {
#                 return;
#             }
# 
#             let (mut l, mut r) = (l + self.size, r + self.size);
# 
#             for i in (1..=self.log).rev() {
#                 if ((l >> i) << i) != l {
#                     self.push(l >> i);
#                 }
#                 if ((r >> i) << i) != r {
#                     self.push((r - 1) >> i);
#                 }
#             }
# 
#             let (l2, r2) = (l, r);
#             while l < r {
#                 if l & 1 == 1 {
#                     self.all_apply(l, f);
#                     l += 1;
#                 }
#                 if r & 1 == 1 {
#                     r -= 1;
#                     self.all_apply(r, f);
#                 }
#                 l >>= 1;
#                 r >>= 1;
#             }
#             l = l2;
#             r = r2;
# 
#             for i in 1..=self.log {
#                 if ((l >> i) << i) != l {
#                     self.update(l >> i);
#                 }
#                 if ((r >> i) << i) != r {
#                     self.update((r - 1) >> i);
#                 }
#             }
#         }
# 
#         pub fn max_right(&mut self, l: usize, g: impl Fn(&S) -> bool) -> usize {
#             if l == self.n {
#                 return self.n;
#             }
# 
#             let mut l = l + self.size;
#             for i in (1..=self.log).rev() {
#                 self.push(l >> i);
#             }
# 
#             let mut sm = S::e();
#             loop {
#                 l >>= l.trailing_zeros();
#                 if !g(&S::opr(&sm, &self.data[l])) {
#                     while l < self.size {
#                         self.push(l);
#                         l <<= 1;
#                         let tmp = S::opr(&sm, &self.data[l]);
#                         if g(&tmp) {
#                             sm = tmp;
#                             l += 1;
#                         }
#                     }
#                     return l - self.size;
#                 }
#                 sm.opr_lhs(&self.data[l]);
#                 l += 1;
# 
#                 if l & ((!l) + 1) != l {
#                     break;
#                 }
#             }
# 
#             self.n
#         }
# 
#         pub fn min_left(&mut self, r: usize, g: impl Fn(&S) -> bool) -> usize {
#             if r == 0 {
#                 return 0;
#             }
# 
#             let mut r = r + self.size;
#             for i in (1..=self.log).rev() {
#                 self.push((r - 1) >> i);
#             }
# 
#             let mut sm = S::e();
#             loop {
#                 r -= 1;
#                 while r > 1 && r & 1 == 1 {
#                     r >>= 1;
#                 }
# 
#                 if !g(&S::opr(&self.data[r], &sm)) {
#                     while r < self.size {
#                         self.push(r);
#                         r = (r << 1) + 1;
#                         let tmp = S::opr(&self.data[r], &sm);
#                         if g(&tmp) {
#                             sm = tmp;
#                             r -= 1;
#                         }
#                     }
#                     return r + 1 - self.size;
#                 }
#                 sm.opr_rhs(&self.data[r]);
# 
#                 if r & ((!r) + 1) == r {
#                     break;
#                 }
#             }
# 
#             0
#         }
#     }
# }
```

## Code

```rust,noplayground
mod lazyseg {
    use std::ops::RangeBounds;

    fn ceil_pow2(n: usize) -> u32 {
        let mut x = 0;
        while 1 << x < n {
            x += 1;
        }
        x
    }

    fn range_to_bounds(n: usize, range: impl RangeBounds<usize>) -> (usize, usize) {
        use std::ops::Bound::*;

        let l = match range.start_bound() {
            Included(&v) => v,
            Excluded(&v) => v + 1,
            Unbounded => 0,
        };

        let r = match range.end_bound() {
            Included(&v) => v + 1,
            Excluded(&v) => v,
            Unbounded => n,
        };

        (l, r)
    }

    pub trait LzMonoid: Sized {
        fn e() -> Self;
        fn opr_lhs(&mut self, rhs: &Self);
        fn opr_rhs(&mut self, lhs: &Self);
        fn opr(lhs: &Self, rhs: &Self) -> Self {
            let mut ret = Self::e();
            ret.opr_rhs(lhs);
            ret.opr_lhs(rhs);
            ret
        }
        fn opr_set(&mut self, lhs: &Self, rhs: &Self) {
            *self = Self::opr(lhs, rhs);
        }
    }

    pub trait LzMap<S>: Sized {
        fn id() -> Self;
        fn map(&self, apply: &mut S);
        fn compos_lhs(&mut self, rhs: &Self);
        fn compos_rhs(&mut self, lhs: &Self);
        fn compos(lhs: &Self, rhs: &Self) -> Self {
            let mut ret = Self::id();
            ret.compos_rhs(lhs);
            ret.compos_lhs(rhs);
            ret
        }
        fn opr_self(&mut self, lhs: &Self, rhs: &Self) {
            *self = Self::compos(lhs, rhs);
        }
    }

    pub struct LazySeg<S, F> {
        n: usize,
        size: usize,
        log: u32,
        data: Vec<S>,
        lazy: Vec<F>,
    }

    impl<S, F> LazySeg<S, F>
    where
        S: LzMonoid,
        F: LzMap<S>,
    {
        fn update(&mut self, k: usize) {
            let val = S::opr(&self.data[k << 1], &self.data[(k << 1) + 1]);
            self.data[k] = val;
        }

        fn all_apply(&mut self, k: usize, f: &F) {
            f.map(&mut self.data[k]);
            if k < self.size {
                self.lazy[k].compos_rhs(f);
            }
        }

        fn all_apply_unsafe(size: usize, k: usize, f: usize, data: &mut [S], lazy: &mut [F]) {
            // f < k
            lazy[f].map(&mut data[k]);
            if k < size {
                let (fs, ks) = lazy.split_at_mut(k);
                ks[0].compos_rhs(&fs[f]);
            }
        }

        fn push(&mut self, k: usize) {
            Self::all_apply_unsafe(self.size, k << 1, k, &mut self.data, &mut self.lazy);
            Self::all_apply_unsafe(self.size, (k << 1) + 1, k, &mut self.data, &mut self.lazy);
            self.lazy[k] = F::id();
        }

        pub fn new(arr: Vec<S>) -> Self {
            let n = arr.len();
            let log = ceil_pow2(n);
            let size = 1 << log;
            let stsize = 1 << (log + 1);

            let mut data = Vec::with_capacity(stsize);
            data.extend((0..size).map(|_| S::e()));
            data.extend(arr.into_iter());
            data.extend((data.len()..stsize).map(|_| S::e()));

            let mut ls = Self {
                n,
                log,
                size,
                data,
                lazy: Vec::from_iter((0..size).map(|_| F::id())),
            };
            for i in (1..size).rev() {
                ls.update(i);
            }
            ls
        }

        pub fn set(&mut self, i: usize, v: S) {
            let i = i + self.size;
            for j in (1..=self.log).rev() {
                self.push(i >> j);
            }
            self.data[i] = v;
            for j in 1..=self.log {
                self.update(i >> j);
            }
        }

        pub fn get(&mut self, i: usize) -> &S {
            let i = i + self.size;
            for j in (1..=self.log).rev() {
                self.push(i >> j);
            }
            &self.data[i]
        }

        pub fn prod(&mut self, range: impl RangeBounds<usize>) -> S {
            let (l, r) = range_to_bounds(self.size, range);

            if l == 0 && r == self.size {
                let mut ret = S::e();
                ret.opr_lhs(&self.data[1]);
                return ret;
            } else if l == r {
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
                    sml.opr_lhs(&self.data[l]);
                    l += 1;
                }
                if r & 1 == 1 {
                    r -= 1;
                    smr.opr_rhs(&self.data[r]);
                }
                l >>= 1;
                r >>= 1;
            }

            S::opr(&sml, &smr)
        }

        pub fn apply(&mut self, i: usize, f: &F) {
            let i = i + self.size;
            for j in (1..=self.log).rev() {
                self.push(i >> j);
            }
            f.map(&mut self.data[i]);
            for j in 1..=self.log {
                self.update(i >> j);
            }
        }

        pub fn apply_range(&mut self, range: impl RangeBounds<usize>, f: &F) {
            let (l, r) = range_to_bounds(self.size, range);

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

        pub fn max_right(&mut self, l: usize, g: impl Fn(&S) -> bool) -> usize {
            if l == self.n {
                return self.n;
            }

            let mut l = l + self.size;
            for i in (1..=self.log).rev() {
                self.push(l >> i);
            }

            let mut sm = S::e();
            loop {
                l >>= l.trailing_zeros();
                if !g(&S::opr(&sm, &self.data[l])) {
                    while l < self.size {
                        self.push(l);
                        l <<= 1;
                        let tmp = S::opr(&sm, &self.data[l]);
                        if g(&tmp) {
                            sm = tmp;
                            l += 1;
                        }
                    }
                    return l - self.size;
                }
                sm.opr_lhs(&self.data[l]);
                l += 1;

                if l & ((!l) + 1) != l {
                    break;
                }
            }

            self.n
        }

        pub fn min_left(&mut self, r: usize, g: impl Fn(&S) -> bool) -> usize {
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

                if !g(&S::opr(&self.data[r], &sm)) {
                    while r < self.size {
                        self.push(r);
                        r = (r << 1) + 1;
                        let tmp = S::opr(&self.data[r], &sm);
                        if g(&tmp) {
                            sm = tmp;
                            r -= 1;
                        }
                    }
                    return r + 1 - self.size;
                }
                sm.opr_rhs(&self.data[r]);

                if r & ((!r) + 1) == r {
                    break;
                }
            }

            0
        }
    }
}
```

## APIs

Will be done soon. For `max_right` and `min_left`, either check the explanation from [the segment tree documentation](./segtree.md#apis), or see the below mathematical definitions (which will be replaced with better explanation).

### `max_right`
Given an index \\(l\\) and a check function \\( f : S \rightarrow bool \\), `max_right` finds an index \\(r\\) such that satisfies both of the following conditions:
- \\(r=l\\) or \\( f \left( A_l \cdot A_{l+1} \cdots A_{r-1} \right) = true \\)
- \\(r=n\\) or \\( f \left( A_l \cdot A_{l+1} \cdots A_r \right) = false \\)

If \\(f\\) is monotone, this is the maximum \\(r\\) that satisfies \\( f \left( A_l \cdot A_{l+1} \cdots A_{r-1} \right) = true \\).

It should be guaranteed that \\(f(e)\\) is true, \\(0 \leq l \leq n\\), and \\(f\\) has no side effects i.e. calling \\(f\\) for the same value should always return the same result.

The search for \\(r\\) is done by binary search, so the time complexity of this function is \\( O(\log{n}) \\).

### `min_left`
Given an index \\(r\\) and a check function \\( f : S \rightarrow bool \\), `max_right` finds an index \\(l\\) such that satisfies both of the following conditions:
- \\(l=r\\) or \\( f \left( A_l \cdot A_{l+1} \cdots A_{r-1} \right) = true \\)
- \\(l=0\\) or \\( f \left( A_{l-1} \cdot A_l \cdots A_{r-1} \right) = false \\)

If \\(f\\) is monotone, this is the minimum \\(l\\) that satisfies \\( f \left( A_l \cdot A_{l+1} \cdots A_{r-1} \right) = true \\).

It should be guaranteed that \\(f(e)\\) is true, \\(0 \leq r \leq n\\), and \\(f\\) has no side effects i.e. calling \\(f\\) for the same value should always return the same result.

The search for \\(l\\) is done by binary search, so the time complexity of this function is \\( O(\log{n}) \\).
