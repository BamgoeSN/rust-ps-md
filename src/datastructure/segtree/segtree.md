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
use segtree::*;

# fn main() {
// Product segment tree with size of 10 and all elements being 1
let st = SegTree::new(10, std::iter::repeat(1), 1, |x, y| x * y % 1000000007);
// Sum segment tree with initial values
let st = SegTree::new(10, vec![1, 3, 2, 4, 3, 5, 4, 6, 5, 7], 0, |x, y| x + y);

// Sum segment tree with size of 10 and all elements being 0
let mut st = SegTree::new(10, None, 0, |x, y| x + y);

st.update(2, |_| 3);
println!("{}", st.get(2)); // 3

let prev = st.get(4);
st.update(4, |x| x + 2);
let curr = st.get(4);
println!("{prev} -> {curr}"); // 0 -> 2

println!("{}", st.prod(..4)); // 3
println!("{}", st.prod(4..)); // 2
println!("{}", st.prod(..)); // 5
println!("{}", st.prod(100..101)); // 0
println!("{}", st.prod(4..2)); // 0

for i in 1..=st.len() {
    print!("{} ", st.prod(1..i));
}
println!(); // 0 0 3 3 5 5 5 5 5 5
println!("{}", st.partition_point(1, |x| x < 3)); // 3
println!("{}", st.partition_point(1, |x| x < 5)); // 5
# }
# 
# mod segtree {
#     use std::ops::RangeBounds;
# 
#     /// A segment tree is a data structure for a monoid type `T`.
#     ///
#     /// Given all the constraints written in the comment of `SegTree::new`, a segment tree can process the following queries in O(TlogN) time assuming `op` all run in O(T).
#     /// - Changing the value of an element
#     /// - Calculating the product of elements in an interval, combined with `op`
#     pub struct SegTree<T, O> {
#         n: usize,
#         data: Vec<T>,
#         e: T,
#         op: O,
#         size: usize,
#         log: u32,
#     }
# 
#     impl<T: Copy, O: Fn(T, T) -> T> SegTree<T, O> {
#         fn get_bounds(&self, range: impl RangeBounds<usize>) -> (usize, usize) {
#             use std::ops::Bound::*;
#             let n = self.len();
#             let l = match range.start_bound() {
#                 Included(&v) => v,
#                 Excluded(&v) => v + 1,
#                 Unbounded => 0,
#             };
#             let r = match range.end_bound() {
#                 Included(&v) => (v + 1).min(n),
#                 Excluded(&v) => v.min(n),
#                 Unbounded => n,
#             };
#             if l > r {
#                 return (l, l);
#             }
#             (l, r)
#         }
# 
#         fn upd(&mut self, k: usize) {
#             self.data[k] = (self.op)(self.data[k * 2], self.data[(k * 2) + 1]);
#         }
# 
#         /// Returns a new segment tree of size `n` built from `iter`.
#         ///
#         /// The meanings of parameters and some generic types are as follows.
#         /// - `T` is a type of values in the array the segment tree represents.
#         /// - `n` is a number of elements in the array.
#         /// - `iter` is an iterator returning initial values of the array.
#         ///   - If `iter.count() < n`, then the rest is filled with `e`.
#         ///   - If `iter.count() > n`, the array is truncated down to the length of `n`.
#         /// - `op: impl Fn(T, T) -> T` is a binary operator for `T`.
#         /// - `e` is an identity for `op`.
#         ///
#         /// The following notations will be used from now on.
#         /// - `op(a, b)` is denoted as `a*b`.
#         ///
#         /// Constraints of parameters are as follows.
#         /// - `op` and `e` must make `T` a monoid. That is, `op` and `e` should be given so that `T` can satisfy the following conditions.
#         ///   - `T` is associative under `op`. That is, `(a*b)*c == a*(b*c)` for all `[a, b, c]: [T; 3]`.
#         ///   - `T` has `e` as an identity element under `op`. That is, `a*e == e*a == a` for all `a: T`.
#         ///
#         /// For example, a generic range sum segment tree with every value initialized with `0` and of length `n` can be constucted as follows.
#         /// ```no_run
#         /// let mut st = SegTree::new(n, None, 0i64, |x, y| x + y);
#         /// ```
#         pub fn new(n: usize, iter: impl IntoIterator<Item = T>, e: T, op: O) -> Self {
#             let size = n.next_power_of_two();
#             let log = size.trailing_zeros();
# 
#             let mut data = vec![e; size];
#             data.extend(iter.into_iter().take(n));
#             data.resize(2 * size, e);
# 
#             let mut st = Self { n, data, e, op, size, log };
#             for i in (1..size).rev() {
#                 st.upd(i);
#             }
#             st
#         }
# 
#         /// Returns the length of the array.
#         pub fn len(&self) -> usize {
#             self.n
#         }
# 
#         /// Returns the `i`-th value of the array.
#         pub fn get(&self, i: usize) -> T {
#             self.data[i + self.size]
#         }
# 
#         /// Assign `upd_to(self.get(i))` to the `i`-th element.
#         pub fn update(&mut self, i: usize, upd_to: impl Fn(T) -> T) {
#             let i = i + self.size;
#             self.data[i] = upd_to(self.data[i]);
#             for j in 1..=self.log {
#                 self.upd(i >> j);
#             }
#         }
# 
#         /// Returns the product of elements in `range`.
#         pub fn prod(&self, range: impl RangeBounds<usize>) -> T {
#             let (mut l, mut r) = self.get_bounds(range);
#             (l += self.size, r += self.size);
# 
#             if (l, r) == (0, self.n) {
#                 return self.data[1];
#             } else if l == r {
#                 return self.e;
#             }
# 
#             let (mut sml, mut smr) = (self.e, self.e);
#             while l < r {
#                 if l & 1 == 1 {
#                     sml = (self.op)(sml, self.data[l]);
#                     l += 1;
#                 }
#                 if r & 1 == 1 {
#                     r -= 1;
#                     smr = (self.op)(self.data[r], smr);
#                 }
#                 (l >>= 1, r >>= 1);
#             }
# 
#             (self.op)(sml, smr)
#         }
# 
#         /// For a function `pred` which has a nonnegative value `x`, such that `pred(self.prod(l..r))` is `false` if and only if `x <= r`, `self.partition_point(l, pred)` returns the value of such `x`.
#         /// That is, this is the minimum value of `r` such that `pred(self.prod(l..r))` starts to be `false`.
#         /// If `pred(self.e)` is `true`, then this function assumes that `pred(self.prod(l..r))` is always `true` for any `r` in range `l..=self.len()` and returns `l`.
#         /// However, it's recommended to always set `pred(self.e)` to be `true` to avoid unnecessary case works.
#         ///
#         /// ## Constraints
#         /// - `0 <= l <= self.len()`
#         ///
#         /// ## Examples
#         /// `f(r) := pred(self.prod(l..r))`
#         ///
#         /// Given that `self.len() == 7`, calling `self.partition_point(0)` returns values written below.
#         /// ```text
#         ///    r |     0     1     2     3     4     5     6     7     8
#         ///
#         /// f(r) |  true  true  true  true false false false false   N/A
#         ///                             returns^
#         ///
#         /// f(r) | false false false false false false false false   N/A
#         ///     returns^
#         ///
#         /// f(r) |  true  true  true  true  true  true  true  true   N/A
#         ///                                                     returns^
#         /// ```
#         pub fn partition_point(&self, l: usize, pred: impl Fn(T) -> bool) -> usize {
#             if !pred(self.e) {
#                 // `pred(self.prod(l..l))` is `false`
#                 // Thus l is returned.
#                 // This case is not covered in the original implementation as it simply requires pred(self.e) to be `true`
#                 return l;
#             }
# 
#             if l == self.n {
#                 // `pred(self.e)` has already been checked that it's `true`.
#                 // Thus the answer must be `self.n`.
#                 return self.n;
#             }
# 
#             let mut l = l + self.size;
#             let mut sm = self.e;
# 
#             loop {
#                 l >>= l.trailing_zeros();
#                 if !pred((self.op)(sm, self.data[l])) {
#                     while l < self.size {
#                         l <<= 1;
#                         let tmp = (self.op)(sm, self.data[l]);
#                         if pred(tmp) {
#                             sm = tmp;
#                             l += 1;
#                         }
#                     }
#                     return l + 1 - self.size;
#                 }
#                 sm = (self.op)(sm, self.data[l]);
#                 l += 1;
#                 if l & ((!l) + 1) == l {
#                     break;
#                 }
#             }
#             self.n + 1
#         }
# 
#         /// For a function `pred` which has a value `x` less than or equal to `r`, such that `pred(self.prod(l..r))` is `true` if and only if `x <= l`, `self.left_partition_point(r, pred)` returns the value of such `x`.
#         /// That is, this is the minimum value of `l` such that `pred(self.prod(l..r))` starts to be `true`.
#         /// If `pred(self.e)` is `false`, then this function assumes that `pred(self.prod(l..r))` is always `false` for any `l` in range `0..=r` and returns `r+1`.
#         /// However, it's recommended to always set `pred(self.e)` to be `true` to avoid unnecessary case works.
#         ///
#         /// ## Constraints
#         /// - `0 <= r <= self.len()`
#         ///
#         /// ## Examples
#         /// `f(l) := pred(self.prod(l..r))`
#         ///
#         /// Calling `self.left_partition_point(7)` returns values written below.
#         /// ```text
#         ///    l |     0     1     2     3     4     5     6     7     8
#         ///
#         /// f(l) | false false false false  true  true  true  true   N/A
#         ///                             returns^
#         ///
#         /// f(l) |  true  true  true  true  true  true  true  true   N/A
#         ///     returns^
#         ///
#         /// f(l) | false false false false false false false false   N/A
#         ///                                                     returns^
#         /// ```
#         pub fn left_partition_point(&self, r: usize, pred: impl Fn(T) -> bool) -> usize {
#             if !pred(self.e) {
#                 return r + 1;
#             }
# 
#             if r == 0 {
#                 // `pred(self.e)` is always `true` at this point
#                 return 0;
#             }
# 
#             let mut r = r + self.size;
#             let mut sm = self.e;
# 
#             loop {
#                 r -= 1;
#                 while r > 1 && r & 1 == 1 {
#                     r >>= 1;
#                 }
#                 if !pred((self.op)(self.data[r], sm)) {
#                     while r < self.size {
#                         r = (r << 1) + 1;
#                         let tmp = (self.op)(self.data[r], sm);
#                         if pred(tmp) {
#                             sm = tmp;
#                             r -= 1;
#                         }
#                     }
#                     return r + 1 - self.size;
#                 }
#                 sm = (self.op)(self.data[r], sm);
#                 if r & ((!r) + 1) == r {
#                     break;
#                 }
#             }
#             0
#         }
#     }
# }
```

## Code

```rust,noplayground
mod segtree {
    use std::ops::RangeBounds;

    /// A segment tree is a data structure for a monoid type `T`.
    ///
    /// Given all the constraints written in the comment of `SegTree::new`, a segment tree can process the following queries in O(TlogN) time assuming `op` all run in O(T).
    /// - Changing the value of an element
    /// - Calculating the product of elements in an interval, combined with `op`
    pub struct SegTree<T, O> {
        n: usize,
        data: Vec<T>,
        e: T,
        op: O,
        size: usize,
        log: u32,
    }

    impl<T: Copy, O: Fn(T, T) -> T> SegTree<T, O> {
        fn get_bounds(&self, range: impl RangeBounds<usize>) -> (usize, usize) {
            use std::ops::Bound::*;
            let n = self.len();
            let l = match range.start_bound() {
                Included(&v) => v,
                Excluded(&v) => v + 1,
                Unbounded => 0,
            };
            let r = match range.end_bound() {
                Included(&v) => (v + 1).min(n),
                Excluded(&v) => v.min(n),
                Unbounded => n,
            };
            if l > r {
                return (l, l);
            }
            (l, r)
        }

        fn upd(&mut self, k: usize) {
            self.data[k] = (self.op)(self.data[k * 2], self.data[(k * 2) + 1]);
        }

        /// Returns a new segment tree of size `n` built from `iter`.
        ///
        /// The meanings of parameters and some generic types are as follows.
        /// - `T` is a type of values in the array the segment tree represents.
        /// - `n` is a number of elements in the array.
        /// - `iter` is an iterator returning initial values of the array.
        ///   - If `iter.count() < n`, then the rest is filled with `e`.
        ///   - If `iter.count() > n`, the array is truncated down to the length of `n`.
        /// - `op: impl Fn(T, T) -> T` is a binary operator for `T`.
        /// - `e` is an identity for `op`.
        ///
        /// The following notations will be used from now on.
        /// - `op(a, b)` is denoted as `a*b`.
        ///
        /// Constraints of parameters are as follows.
        /// - `op` and `e` must make `T` a monoid. That is, `op` and `e` should be given so that `T` can satisfy the following conditions.
        ///   - `T` is associative under `op`. That is, `(a*b)*c == a*(b*c)` for all `[a, b, c]: [T; 3]`.
        ///   - `T` has `e` as an identity element under `op`. That is, `a*e == e*a == a` for all `a: T`.
        ///
        /// For example, a generic range sum segment tree with every value initialized with `0` and of length `n` can be constucted as follows.
        /// ```no_run
        /// let mut st = SegTree::new(n, None, 0i64, |x, y| x + y);
        /// ```
        pub fn new(n: usize, iter: impl IntoIterator<Item = T>, e: T, op: O) -> Self {
            let size = n.next_power_of_two();
            let log = size.trailing_zeros();

            let mut data = vec![e; size];
            data.extend(iter.into_iter().take(n));
            data.resize(2 * size, e);

            let mut st = Self { n, data, e, op, size, log };
            for i in (1..size).rev() {
                st.upd(i);
            }
            st
        }

        /// Returns the length of the array.
        pub fn len(&self) -> usize {
            self.n
        }

        /// Returns the `i`-th value of the array.
        pub fn get(&self, i: usize) -> T {
            self.data[i + self.size]
        }

        /// Assign `upd_to(self.get(i))` to the `i`-th element.
        pub fn update(&mut self, i: usize, upd_to: impl Fn(T) -> T) {
            let i = i + self.size;
            self.data[i] = upd_to(self.data[i]);
            for j in 1..=self.log {
                self.upd(i >> j);
            }
        }

        /// Returns the product of elements in `range`.
        pub fn prod(&self, range: impl RangeBounds<usize>) -> T {
            let (mut l, mut r) = self.get_bounds(range);
            (l += self.size, r += self.size);

            if (l, r) == (0, self.n) {
                return self.data[1];
            } else if l == r {
                return self.e;
            }

            let (mut sml, mut smr) = (self.e, self.e);
            while l < r {
                if l & 1 == 1 {
                    sml = (self.op)(sml, self.data[l]);
                    l += 1;
                }
                if r & 1 == 1 {
                    r -= 1;
                    smr = (self.op)(self.data[r], smr);
                }
                (l >>= 1, r >>= 1);
            }

            (self.op)(sml, smr)
        }

        /// For a function `pred` which has a nonnegative value `x`, such that `pred(self.prod(l..r))` is `false` if and only if `x <= r`, `self.partition_point(l, pred)` returns the value of such `x`.
        /// That is, this is the minimum value of `r` such that `pred(self.prod(l..r))` starts to be `false`.
        /// If `pred(self.e)` is `true`, then this function assumes that `pred(self.prod(l..r))` is always `true` for any `r` in range `l..=self.len()` and returns `l`.
        /// However, it's recommended to always set `pred(self.e)` to be `true` to avoid unnecessary case works.
        ///
        /// ## Constraints
        /// - `0 <= l <= self.len()`
        ///
        /// ## Examples
        /// `f(r) := pred(self.prod(l..r))`
        ///
        /// Given that `self.len() == 7`, calling `self.partition_point(0)` returns values written below.
        /// ```text
        ///    r |     0     1     2     3     4     5     6     7     8
        ///
        /// f(r) |  true  true  true  true false false false false   N/A
        ///                             returns^
        ///
        /// f(r) | false false false false false false false false   N/A
        ///     returns^
        ///
        /// f(r) |  true  true  true  true  true  true  true  true   N/A
        ///                                                     returns^
        /// ```
        pub fn partition_point(&self, l: usize, pred: impl Fn(T) -> bool) -> usize {
            if !pred(self.e) {
                // `pred(self.prod(l..l))` is `false`
                // Thus l is returned.
                // This case is not covered in the original implementation as it simply requires pred(self.e) to be `true`
                return l;
            }

            if l == self.n {
                // `pred(self.e)` has already been checked that it's `true`.
                // Thus the answer must be `self.n`.
                return self.n;
            }

            let mut l = l + self.size;
            let mut sm = self.e;

            loop {
                l >>= l.trailing_zeros();
                if !pred((self.op)(sm, self.data[l])) {
                    while l < self.size {
                        l <<= 1;
                        let tmp = (self.op)(sm, self.data[l]);
                        if pred(tmp) {
                            sm = tmp;
                            l += 1;
                        }
                    }
                    return l + 1 - self.size;
                }
                sm = (self.op)(sm, self.data[l]);
                l += 1;
                if l & ((!l) + 1) == l {
                    break;
                }
            }
            self.n + 1
        }

        /// For a function `pred` which has a value `x` less than or equal to `r`, such that `pred(self.prod(l..r))` is `true` if and only if `x <= l`, `self.left_partition_point(r, pred)` returns the value of such `x`.
        /// That is, this is the minimum value of `l` such that `pred(self.prod(l..r))` starts to be `true`.
        /// If `pred(self.e)` is `false`, then this function assumes that `pred(self.prod(l..r))` is always `false` for any `l` in range `0..=r` and returns `r+1`.
        /// However, it's recommended to always set `pred(self.e)` to be `true` to avoid unnecessary case works.
        ///
        /// ## Constraints
        /// - `0 <= r <= self.len()`
        ///
        /// ## Examples
        /// `f(l) := pred(self.prod(l..r))`
        ///
        /// Calling `self.left_partition_point(7)` returns values written below.
        /// ```text
        ///    l |     0     1     2     3     4     5     6     7     8
        ///
        /// f(l) | false false false false  true  true  true  true   N/A
        ///                             returns^
        ///
        /// f(l) |  true  true  true  true  true  true  true  true   N/A
        ///     returns^
        ///
        /// f(l) | false false false false false false false false   N/A
        ///                                                     returns^
        /// ```
        pub fn left_partition_point(&self, r: usize, pred: impl Fn(T) -> bool) -> usize {
            if !pred(self.e) {
                return r + 1;
            }

            if r == 0 {
                // `pred(self.e)` is always `true` at this point
                return 0;
            }

            let mut r = r + self.size;
            let mut sm = self.e;

            loop {
                r -= 1;
                while r > 1 && r & 1 == 1 {
                    r >>= 1;
                }
                if !pred((self.op)(self.data[r], sm)) {
                    while r < self.size {
                        r = (r << 1) + 1;
                        let tmp = (self.op)(self.data[r], sm);
                        if pred(tmp) {
                            sm = tmp;
                            r -= 1;
                        }
                    }
                    return r + 1 - self.size;
                }
                sm = (self.op)(self.data[r], sm);
                if r & ((!r) + 1) == r {
                    break;
                }
            }
            0
        }
    }
}
```

---

Last modified on 231007