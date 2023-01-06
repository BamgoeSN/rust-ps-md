# Segment Tree

Reference: AtCoder library <https://atcoder.github.io/ac-library/production/document_en/index.html>

A segment tree is a data structure for monoids \\( (S, \cdot : S \times S \rightarrow S, e \in S) \\). A monoid is an algebraic structure which follows the following conditions:
- \\(\cdot\\) is associative. That is, \\( (a \cdot b) \cdot c = a \cdot (b \cdot c) \\) for all \\( a, b, c \in S \\).
- There is the identity element \\(e\\) such that \\( a \cdot e = e \cdot a = a \\) for all \\( a \in S \\).
  
Given an array \\(A\\) of length \\(n\\) consists of the monoid \\(S\\) as described above, a segment tree on it can process the following queries in \\(O (\log{n})\\) time:
- Update an element
- Calculate the product of the elements of an interval

assuming that calculating the product of two elements takes \\(O(1)\\) time.

## Example
```rust
# use segtree::*;
# 
// Interval sum segment tree
impl Monoid for i32 {
    fn e() -> Self { 0 }
    fn opr_lhs(&mut self, rhs: &Self) { *self += rhs; }
    fn opr_rhs(&mut self, lhs: &Self) { *self += lhs; }
}

# fn main() {
let mut st = SegTree::new(vec![0i32, 4, 0, 0, 0, 0, 0, 0, 0]);
st.set(2, 5);
st.set(4, 8);

println!("{}", st[2]);         // 5
println!("{}", st.prod(0..3)); // 9
println!("{}", st.prod(..));   // 17

let r = st.max_right(2, |&x| x < 13);
println!("{}", r);             // 4
let l = st.min_left(st.len(), |&x| x < 100);
println!("{}", l);             // 0
# }
# 
# mod segtree {
#     use std::ops::{Index, RangeBounds};
# 
#     fn ceil_pow2(n: usize) -> u32 {
#         if n == 0 {
#             return 0;
#         }
#         let dig = usize::BITS - n.leading_zeros();
#         let x = 1 << (dig - 1);
#         if n == x {
#             dig - 1
#         } else {
#             dig
#         }
#     }
# 
#     pub trait Monoid: Sized {
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
#     pub struct SegTree<S> {
#         n: usize,
#         size: usize,
#         log: u32,
#         data: Vec<S>,
#     }
# 
#     impl<S: Monoid> SegTree<S> {
#         fn update(&mut self, k: usize) {
#             let mut ret = S::e();
#             ret.opr_set(&self.data[k << 1], &self.data[(k << 1) + 1]);
#             self.data[k] = ret;
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
#             let mut st = Self { n, size, log, data };
#             for i in (1..size).rev() {
#                 st.update(i);
#             }
#             st
#         }
# 
#         pub fn len(&self) -> usize {
#             self.n
#         }
# 
#         pub fn set(&mut self, i: usize, v: S) {
#             let i = i + self.size;
#             self.data[i] = v;
#             for j in 1..=self.log {
#                 self.update(i >> j);
#             }
#         }
# 
#         pub fn get(&self, i: usize) -> &S {
#             &self.data[i + self.size]
#         }
# 
#         pub fn prod(&self, range: impl RangeBounds<usize>) -> S {
#             use std::ops::Bound::*;
#             let (mut sml, mut smr) = (S::e(), S::e());
#             let mut l = match range.start_bound() {
#                 Included(&v) => v,
#                 Excluded(&v) => v - 1,
#                 Unbounded => 0,
#             } + self.size;
#             let mut r = match range.end_bound() {
#                 Included(&v) => v + 1,
#                 Excluded(&v) => v,
#                 Unbounded => self.n,
#             } + self.size;
# 
#             if (l, r) == (0, self.n) {
#                 let mut ret = S::e();
#                 ret.opr_lhs(&self.data[1]);
#                 return ret;
#             }
# 
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
#             sml.opr_lhs(&smr);
#             sml
#         }
# 
#         /// Given an index l and a check function f, max_right finds an index r that satisfies
#         ///   (r == l || f(self.prod(l..r)) && (r == n || !f(self.prod(l..=r)))
#         /// If f is monotone, this is the maximum r that satisfies f(self.prod(l..r)).
#         /// It should be guaranteed that f(S::e()) is true, 0 <= l and l <= n.
#         pub fn max_right(&self, l: usize, f: impl Fn(&S) -> bool) -> usize {
#             if l == self.n {
#                 return self.n;
#             }
# 
#             let mut l = l + self.size;
#             let mut sm = S::e();
# 
#             loop {
#                 l >>= l.trailing_zeros();
#                 if !f(&S::opr(&sm, &self.data[l])) {
#                     while l < self.size {
#                         l <<= 1;
#                         let tmp = S::opr(&sm, &self.data[l]);
#                         if f(&tmp) {
#                             sm = tmp;
#                             l += 1;
#                         }
#                     }
#                     return l - self.size;
#                 }
#                 sm.opr_lhs(&self.data[l]);
#                 l += 1;
# 
#                 if l & ((!l) + 1) == l {
#                     break;
#                 }
#             }
#             self.n
#         }
# 
#         /// Given an index r and a check function f, min_left finds an index l that satisfies
#         ///   (l == r || f(self.prod(l..r))) && (l == 0 || !f(self.prod(l-1..r)))
#         /// If f is monotone, this is the minimum l that satisfies f(self.prod(l..r)).
#         /// It should be guaranteed that f(S::e()) is true, 0 <= r and r <= n.
#         pub fn min_left(&self, r: usize, f: impl Fn(&S) -> bool) -> usize {
#             if r == 0 {
#                 return 0;
#             }
# 
#             let mut r = r + self.size;
#             let mut sm = S::e();
# 
#             loop {
#                 r -= 1;
#                 while r > 1 && r & 1 == 1 {
#                     r >>= 1;
#                 }
#                 if !f(&S::opr(&self.data[r], &sm)) {
#                     while r < self.size {
#                         r = (r << 1) + 1;
#                         let tmp = S::opr(&self.data[r], &sm);
#                         if f(&tmp) {
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
#             0
#         }
#     }
# 
#     impl<S: Monoid> Index<usize> for SegTree<S> {
#         type Output = S;
#         fn index(&self, index: usize) -> &Self::Output {
#             self.get(index)
#         }
#     }
# }
```

## Code

```rust,noplayground
mod segtree {
    use std::ops::{Index, RangeBounds};

    fn ceil_pow2(n: usize) -> u32 {
        if n == 0 {
            return 0;
        }
        let dig = usize::BITS - n.leading_zeros();
        let x = 1 << (dig - 1);
        if n == x {
            dig - 1
        } else {
            dig
        }
    }

    pub trait Monoid: Sized {
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

    pub struct SegTree<S> {
        n: usize,
        size: usize,
        log: u32,
        data: Vec<S>,
    }

    impl<S: Monoid> SegTree<S> {
        fn update(&mut self, k: usize) {
            let mut ret = S::e();
            ret.opr_set(&self.data[k << 1], &self.data[(k << 1) + 1]);
            self.data[k] = ret;
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

            let mut st = Self { n, size, log, data };
            for i in (1..size).rev() {
                st.update(i);
            }
            st
        }

        pub fn len(&self) -> usize {
            self.n
        }

        pub fn get(&self, i: usize) -> &S {
            &self.data[i + self.size]
        }

        pub fn set(&mut self, i: usize, v: S) {
            let i = i + self.size;
            self.data[i] = v;
            for j in 1..=self.log {
                self.update(i >> j);
            }
        }

        pub fn prod(&self, range: impl RangeBounds<usize>) -> S {
            use std::ops::Bound::*;
            let (mut sml, mut smr) = (S::e(), S::e());
            let mut l = match range.start_bound() {
                Included(&v) => v,
                Excluded(&v) => v - 1,
                Unbounded => 0,
            } + self.size;
            let mut r = match range.end_bound() {
                Included(&v) => v + 1,
                Excluded(&v) => v,
                Unbounded => self.n,
            } + self.size;

            if (l, r) == (0, self.n) {
                let mut ret = S::e();
                ret.opr_lhs(&self.data[1]);
                return ret;
            }

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

            sml.opr_lhs(&smr);
            sml
        }

        /// Given an index l and a check function f, max_right finds an index r that satisfies
        ///   (r == l || f(self.prod(l..r)) && (r == n || !f(self.prod(l..=r)))
        /// If f is monotone, this is the maximum r that satisfies f(self.prod(l..r)).
        /// It should be guaranteed that f(S::e()) is true, 0 <= l and l <= n.
        pub fn max_right(&self, l: usize, f: impl Fn(&S) -> bool) -> usize {
            if l == self.n {
                return self.n;
            }

            let mut l = l + self.size;
            let mut sm = S::e();

            loop {
                l >>= l.trailing_zeros();
                if !f(&S::opr(&sm, &self.data[l])) {
                    while l < self.size {
                        l <<= 1;
                        let tmp = S::opr(&sm, &self.data[l]);
                        if f(&tmp) {
                            sm = tmp;
                            l += 1;
                        }
                    }
                    return l - self.size;
                }
                sm.opr_lhs(&self.data[l]);
                l += 1;

                if l & ((!l) + 1) == l {
                    break;
                }
            }
            self.n
        }

        /// Given an index r and a check function f, min_left finds an index l that satisfies
        ///   (l == r || f(self.prod(l..r))) && (l == 0 || !f(self.prod(l-1..r)))
        /// If f is monotone, this is the minimum l that satisfies f(self.prod(l..r)).
        /// It should be guaranteed that f(S::e()) is true, 0 <= r and r <= n.
        pub fn min_left(&self, r: usize, f: impl Fn(&S) -> bool) -> usize {
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
                if !f(&S::opr(&self.data[r], &sm)) {
                    while r < self.size {
                        r = (r << 1) + 1;
                        let tmp = S::opr(&self.data[r], &sm);
                        if f(&tmp) {
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

    impl<S: Monoid> Index<usize> for SegTree<S> {
        type Output = S;
        fn index(&self, index: usize) -> &Self::Output {
            self.get(index)
        }
    }
}
```

## APIs

- `trait Monoid`
Represents a monoid \\(S\\) explained above. Three methods `e`, `opr_lhs`, `opr_rhs` are required to be implemented. Implementing `opr` and `opr_set` is optional.
  + `fn e() -> Self` returns an identity element of the monoid.
  + `fn opr_lhs(&mut self, rhs: &Self)` calculates the product of `self` and `rhs` in this order, and sets `self` with it.
  + `fn opr_rhs(&mut self, lhs: &Self)` calculates the product of `lhs` and `self` in this order, and sets `self` with it.
  + (Optional) `fn opr(lhs: &Self, rhs: &Self)` calculates the product of `lhs` and `rhs` and returns it.
  + (Optional) `fn opr_set(&mut self, lhs: &Self, rhs: &Self)` calculates the product of `lhs` and `rhs` and sets `self` with it.
- `fn new(arr: Vec<S>) -> Self` generates a segment tree from `arr`.
- `fn len(&self) -> usize` returns the length of the segment tree, which is equal to the length of the array used for constructing the segment tree.
- `fn get(&self, i: usize) -> &S` returns the reference to the `i`th element of the segment tree.
- `fn set(&mut self, i: usize, v: S)` sets the `i`th element of the segment tree with `v`.
- `fn prod(&self, range: impl RangeBounds<usize>) -> S` returns the product of values of the segment tree within the given range.
  + Example: `let v = st.prod(3..10);` `let u = st.prod(..7);`

- `fn max_right(&self, l: usize, f: impl Fn(&S) -> bool) -> usize` returns an index `r` such that `(r == l || f(self.prod(l..r)) && (r == n || !f(self.prod(l..=r)))`. If `f` is monotone, this is the maximum `r` that makes `f(self.prod(l..r)` true. It must be guaranteed that `f(S::e())` is true, and `0 <= l <= n`. This method is basically equivalent to `partition_point` of a slice type, but with a set left bound.

- `fn min_left(&self, r: usize, f: impl Fn(&S) -> bool) -> usize` returns an index `l` such that `(l == r || f(self.prod(l..r))) && (l == 0 || !f(self.prod(l-1..r)))`. If `f` is monotone, this is the minimum `l` that makes `f(self.prod(l..r))` true. It must be guaranteed that `f(S::e())` is true, and `0 <= r <= n`.
