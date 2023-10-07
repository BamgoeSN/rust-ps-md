# Lazy Segment Tree

Reference: AtCoder library <https://atcoder.github.io/ac-library/production/document_en/index.html>

A lazy segment tree is a data struture for a pair of a [monoid](./segtree.md) \\( (T, \cdot : T \times T \rightarrow T, e \in T) \\) and a set \\(F\\) of \\(T \rightarrow T\\) mappings that satisfies the following properties:
- \\(F\\) contains the identity mapping \\(Id\\) such that \\( Id(x) = x \\) for all \\(x\in T\\).
- \\(F\\) is closed under composition. That is, \\( f \circ g \in F \\) for all \\( f, g \in F \\).
- \\( f (x \cdot y) = f(x) \cdot f(y) \\) hold for all \\(f \in F \\) and \\( x, y \in T \\).

Given an array \\(A\\) of length \\(n\\) consists of the monoid \\(T\\) as described above, a segment tree on it can process the following queries in \\(O (\log{n})\\) time:
- Apply the mapping \\( f \in F \\) on all the elements of an interval
- Calculate the product of the elements of an interval

assuming that calculating the product of two elements takes \\(O(1)\\) time.

## Example

```rust
use lazyseg::*;

# fn main() {
// Generic range addition, range sum lazy segment tree
let mut ls = LazySeg::new(
    10,
    (0..10).map(|i| (i, 1)),
    |(x, l), (y, m)| (x + y, l + m),
    (0i64, 0i64),
    |a, (x, l)| (x + a * l, l),
    |a, b| a + b,
    0i64,
);

println!("{}", ls.prod(2..8).0); // 27
ls.apply_range(3..6, 3);
println!("{}", ls.prod(2..8).0); // 36

for r in 3..=10 {
    print!("{} ", ls.prod(3..r).0);
}
println!(); // 0 6 13 21 27 34 42 51
println!("{}", ls.partition_point(3, |(x, _)| x < 40)); // 9

for l in 0..=7 {
    print!("{} ", ls.prod(l..7).0);
}
println!(); // 30 30 29 27 21 14 6 0
println!("{}", ls.left_partition_point(7, |(x, _)| x < 25)); // 4
# }
# 
# mod lazyseg {
#     use std::ops::RangeBounds;
# 
#     /// A lazy segment tree is a data structure for a monoid type `T` and a mapping `F` that maps a `T` value to another `T` value.
#     ///
#     /// Given all the constraints written in the comments of `LazySeg::new`, a lazy segment tree can process the following queries in O(TlogN) time assuming `op`, `map`, `compos` all run in O(T).
#     /// - Applying the map `f: U` onto all the elements in an interval
#     /// - Calculating the product of elements in an interval, combined with `op`
#     pub struct LazySeg<T, O, F, M, C> {
#         n: usize,
#         data: Vec<T>,
#         lazy: Vec<F>,
#         e: T,
#         op: O,
#         id: F,
#         map: M,
#         compos: C,
#         size: usize,
#         log: u32,
#     }
# 
#     impl<T, O, F, M, C> LazySeg<T, O, F, M, C>
#     where
#         T: Copy,
#         O: Fn(T, T) -> T,
#         F: Copy,
#         M: Fn(F, T) -> T,
#         C: Fn(F, F) -> F,
#     {
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
#             self.data[k] = (self.op)(self.data[k * 2], self.data[k * 2 + 1]);
#         }
# 
#         fn all_apply(&mut self, k: usize, f: F) {
#             self.data[k] = (self.map)(f, self.data[k]);
#             if k < self.size {
#                 self.lazy[k] = (self.compos)(f, self.lazy[k]);
#             }
#         }
# 
#         fn push(&mut self, k: usize) {
#             self.all_apply(k * 2, self.lazy[k]);
#             self.all_apply(k * 2 + 1, self.lazy[k]);
#             self.lazy[k] = self.id;
#         }
# 
#         /// Returns a new lazy segment tree of size `n` built from `iter`.
#         ///
#         /// The meanings of parameters and some generic types are as follows.
#         /// - `T` is a type of values in the array the lazy segment tree represents.
#         /// - `F` is a type of mappings for the array.
#         /// - `n` is a number of elements in the array.
#         /// - `iter` is an iterator returning initial values of the array.
#         ///   - If `iter.count() < n`, then the rest is filled with `e`.
#         ///   - If `iter.count() > n`, the array is truncated down to the length of `n`.
#         /// - `op: impl Fn(T, T) -> T` is a binary operator for `T`.
#         /// - `e` is an identity for `op`.
#         /// - `map: impl Fn(F, T) -> T` defines how to map `T` to another `T` based on the `F` value.
#         /// - `compos: impl Fn(F, F) -> F` defines how to compose two `F`'s.
#         /// - `id` defines an identity for `compos`.
#         ///
#         /// The following notations will be used from now on.
#         /// - `op(a, b)` is denoted as `a*b`.
#         /// - `map(f, a)` is denoted as `f.a`.
#         /// - `map(g, map(f, a))` is denoted as `g.f.a`.
#         ///
#         /// Constraints of parameters are as follows.
#         /// - `op` and `e` must make `T` a monoid. That is, `op` and `e` should be given so that `T` can satisfy the following conditions.
#         ///   - `T` is associative under `op`. That is, `(a*b)*c == a*(b*c)` for all `[a, b, c]: [T; 3]`.
#         ///   - `T` has `e` as an identity element under `op`. That is, `a*e == e*a == a` for all `a: T`.
#         /// - `map`, `compos`, and `id` must satisfy the following conditions.
#         ///   - `compos` should be properly defined. That is, if `compos(g, f) == h`, then `g.f.a == h.a` must hold.
#         ///   - `id` must be a proper identity for `F` under `compos`. That is, `f.id.a == id.f.a == f.a` for all `a: T` and `f: F`.
#         ///   - IMPORTANT: `map` must satisfy `f.(x*y) == f.x * f.y`.
#         ///
#         /// For example, a generic range addition range sum lazy segment tree with every value initialized with `0` and of length `n` can be constructed as follows.
#         /// ```no_run
#         /// let mut ls = LazySeg::new(
#         ///     n,
#         ///     (0..n).map(|_| (0, 1)),
#         ///     |(x, l), (y, m)| (x + y, l + m),
#         ///     (0i64, 0i64),
#         ///     |a, (x, l)| (x + a * l, l),
#         ///     |a, b| a + b,
#         ///     0i64,
#         /// );
#         /// ```
#         /// A so-called "ax+b" lazy segment tree starting with an array of `vec![0; n]` can be constructed as follows.
#         /// ```no_run
#         /// let mut ls = LazySeg::new(
#         ///     n,
#         ///     (0..n).map(|_| (0, 1)),
#         ///     |(x, l), (y, m)| (x + y, l + m),
#         ///     (0i64, 0i64),
#         ///     |(a, b), (x, l)| (a * x, b * l),
#         ///     |(a, b), (c, d)| (a * c, a * d + b),
#         ///     (1i64, 0i64),
#         /// );
#         /// ```
#         pub fn new(n: usize, iter: impl IntoIterator<Item = T>, op: O, e: T, map: M, compos: C, id: F) -> Self {
#             let size = n.next_power_of_two();
#             let log = size.trailing_zeros();
# 
#             let mut data = vec![e; size];
#             data.extend(iter.into_iter().take(n));
#             data.resize(2 * size, e);
# 
#             let mut ls = Self {
#                 n,
#                 data,
#                 lazy: vec![id; size],
#                 e,
#                 op,
#                 id,
#                 map,
#                 compos,
#                 size,
#                 log,
#             };
#             for i in (1..size).rev() {
#                 ls.upd(i);
#             }
#             ls
#         }
# 
#         /// Returns the length of the array.
#         pub fn len(&self) -> usize {
#             self.n
#         }
# 
#         /// Returns the `i`-th value of the array.
#         pub fn get(&mut self, i: usize) -> T {
#             let i = i + self.size;
#             for j in (1..=self.log).rev() {
#                 self.push(i >> j);
#             }
#             self.data[i]
#         }
# 
#         /// Assign `upd_to(self.get(i))` to the `i`-th element.
#         pub fn update(&mut self, i: usize, upd_to: impl Fn(T) -> T) {
#             let i = i + self.size;
#             for j in (1..=self.log).rev() {
#                 self.push(i >> j);
#             }
#             self.data[i] = upd_to(self.data[i]);
#             for j in 1..=self.log {
#                 self.upd(i >> j);
#             }
#         }
# 
#         /// Returns the product of elements in `range`.
#         pub fn prod(&mut self, range: impl RangeBounds<usize>) -> T {
#             let (l, r) = self.get_bounds(range);
# 
#             if l == 0 && r == self.size {
#                 let ret = (self.op)(self.e, self.data[1]);
#                 return ret;
#             } else if l == r {
#                 return self.e;
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
#             (self.op)(sml, smr)
#         }
# 
#         /// Changes every element `x` in `range` of the array to `map.x`.
#         pub fn apply_range(&mut self, range: impl RangeBounds<usize>, map: F) {
#             let (l, r) = self.get_bounds(range);
#             if l >= r {
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
#                     self.all_apply(l, map);
#                     l += 1;
#                 }
#                 if r & 1 == 1 {
#                     r -= 1;
#                     self.all_apply(r, map);
#                 }
#                 l >>= 1;
#                 r >>= 1;
#             }
#             l = l2;
#             r = r2;
# 
#             for i in 1..=self.log {
#                 if ((l >> i) << i) != l {
#                     self.upd(l >> i);
#                 }
#                 if ((r >> i) << i) != r {
#                     self.upd((r - 1) >> i);
#                 }
#             }
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
#         pub fn partition_point(&mut self, l: usize, pred: impl Fn(T) -> bool) -> usize {
#             if !pred(self.e) {
#                 return l;
#             }
# 
#             if l == self.n {
#                 // `pred(self.e)` has already been checked that it's `true`.
#                 return self.n;
#             }
# 
#             let mut l = l + self.size;
#             for i in (1..=self.log).rev() {
#                 self.push(l >> i);
#             }
#             let mut sm = self.e;
# 
#             loop {
#                 l >>= l.trailing_zeros();
#                 if !pred((self.op)(sm, self.data[l])) {
#                     while l < self.size {
#                         self.push(l);
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
#         pub fn left_partition_point(&mut self, r: usize, pred: impl Fn(T) -> bool) -> usize {
#             if !pred(self.e) {
#                 return r + 1;
#             }
# 
#             if r == 0 {
#                 // `pred(self.e)` has already been checked that it's `true`.
#                 return 0;
#             }
# 
#             let mut r = r + self.size;
#             for i in (1..=self.log).rev() {
#                 self.push((r - 1) >> i);
#             }
# 
#             let mut sm = self.e;
#             loop {
#                 r -= 1;
#                 while r > 1 && r & 1 == 1 {
#                     r >>= 1;
#                 }
# 
#                 if !pred((self.op)(self.data[r], sm)) {
#                     while r < self.size {
#                         self.push(r);
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
mod lazyseg {
    use std::ops::RangeBounds;

    /// A lazy segment tree is a data structure for a monoid type `T` and a mapping `F` that maps a `T` value to another `T` value.
    ///
    /// Given all the constraints written in the comments of `LazySeg::new`, a lazy segment tree can process the following queries in O(TlogN) time assuming `op`, `map`, `compos` all run in O(T).
    /// - Applying the map `f: U` onto all the elements in an interval
    /// - Calculating the product of elements in an interval, combined with `op`
    pub struct LazySeg<T, O, F, M, C> {
        n: usize,
        data: Vec<T>,
        lazy: Vec<F>,
        e: T,
        op: O,
        id: F,
        map: M,
        compos: C,
        size: usize,
        log: u32,
    }

    impl<T, O, F, M, C> LazySeg<T, O, F, M, C>
    where
        T: Copy,
        O: Fn(T, T) -> T,
        F: Copy,
        M: Fn(F, T) -> T,
        C: Fn(F, F) -> F,
    {
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
            self.data[k] = (self.op)(self.data[k * 2], self.data[k * 2 + 1]);
        }

        fn all_apply(&mut self, k: usize, f: F) {
            self.data[k] = (self.map)(f, self.data[k]);
            if k < self.size {
                self.lazy[k] = (self.compos)(f, self.lazy[k]);
            }
        }

        fn push(&mut self, k: usize) {
            self.all_apply(k * 2, self.lazy[k]);
            self.all_apply(k * 2 + 1, self.lazy[k]);
            self.lazy[k] = self.id;
        }

        /// Returns a new lazy segment tree of size `n` built from `iter`.
        ///
        /// The meanings of parameters and some generic types are as follows.
        /// - `T` is a type of values in the array the lazy segment tree represents.
        /// - `F` is a type of mappings for the array.
        /// - `n` is a number of elements in the array.
        /// - `iter` is an iterator returning initial values of the array.
        ///   - If `iter.count() < n`, then the rest is filled with `e`.
        ///   - If `iter.count() > n`, the array is truncated down to the length of `n`.
        /// - `op: impl Fn(T, T) -> T` is a binary operator for `T`.
        /// - `e` is an identity for `op`.
        /// - `map: impl Fn(F, T) -> T` defines how to map `T` to another `T` based on the `F` value.
        /// - `compos: impl Fn(F, F) -> F` defines how to compose two `F`'s.
        /// - `id` defines an identity for `compos`.
        ///
        /// The following notations will be used from now on.
        /// - `op(a, b)` is denoted as `a*b`.
        /// - `map(f, a)` is denoted as `f.a`.
        /// - `map(g, map(f, a))` is denoted as `g.f.a`.
        ///
        /// Constraints of parameters are as follows.
        /// - `op` and `e` must make `T` a monoid. That is, `op` and `e` should be given so that `T` can satisfy the following conditions.
        ///   - `T` is associative under `op`. That is, `(a*b)*c == a*(b*c)` for all `[a, b, c]: [T; 3]`.
        ///   - `T` has `e` as an identity element under `op`. That is, `a*e == e*a == a` for all `a: T`.
        /// - `map`, `compos`, and `id` must satisfy the following conditions.
        ///   - `compos` should be properly defined. That is, if `compos(g, f) == h`, then `g.f.a == h.a` must hold.
        ///   - `id` must be a proper identity for `F` under `compos`. That is, `f.id.a == id.f.a == f.a` for all `a: T` and `f: F`.
        ///   - IMPORTANT: `map` must satisfy `f.(x*y) == f.x * f.y`.
        ///
        /// For example, a generic range addition range sum lazy segment tree with every value initialized with `0` and of length `n` can be constructed as follows.
        /// ```no_run
        /// let mut ls = LazySeg::new(
        ///     n,
        ///     (0..n).map(|_| (0, 1)),
        ///     |(x, l), (y, m)| (x + y, l + m),
        ///     (0i64, 0i64),
        ///     |a, (x, l)| (x + a * l, l),
        ///     |a, b| a + b,
        ///     0i64,
        /// );
        /// ```
        /// A so-called "ax+b" lazy segment tree starting with an array of `vec![0; n]` can be constructed as follows.
        /// ```no_run
        /// let mut ls = LazySeg::new(
        ///     n,
        ///     (0..n).map(|_| (0, 1)),
        ///     |(x, l), (y, m)| (x + y, l + m),
        ///     (0i64, 0i64),
        ///     |(a, b), (x, l)| (a * x, b * l),
        ///     |(a, b), (c, d)| (a * c, a * d + b),
        ///     (1i64, 0i64),
        /// );
        /// ```
        pub fn new(n: usize, iter: impl IntoIterator<Item = T>, op: O, e: T, map: M, compos: C, id: F) -> Self {
            let size = n.next_power_of_two();
            let log = size.trailing_zeros();

            let mut data = vec![e; size];
            data.extend(iter.into_iter().take(n));
            data.resize(2 * size, e);

            let mut ls = Self {
                n,
                data,
                lazy: vec![id; size],
                e,
                op,
                id,
                map,
                compos,
                size,
                log,
            };
            for i in (1..size).rev() {
                ls.upd(i);
            }
            ls
        }

        /// Returns the length of the array.
        pub fn len(&self) -> usize {
            self.n
        }

        /// Returns the `i`-th value of the array.
        pub fn get(&mut self, i: usize) -> T {
            let i = i + self.size;
            for j in (1..=self.log).rev() {
                self.push(i >> j);
            }
            self.data[i]
        }

        /// Assign `upd_to(self.get(i))` to the `i`-th element.
        pub fn update(&mut self, i: usize, upd_to: impl Fn(T) -> T) {
            let i = i + self.size;
            for j in (1..=self.log).rev() {
                self.push(i >> j);
            }
            self.data[i] = upd_to(self.data[i]);
            for j in 1..=self.log {
                self.upd(i >> j);
            }
        }

        /// Returns the product of elements in `range`.
        pub fn prod(&mut self, range: impl RangeBounds<usize>) -> T {
            let (l, r) = self.get_bounds(range);

            if l == 0 && r == self.size {
                let ret = (self.op)(self.e, self.data[1]);
                return ret;
            } else if l == r {
                return self.e;
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

        /// Changes every element `x` in `range` of the array to `map.x`.
        pub fn apply_range(&mut self, range: impl RangeBounds<usize>, map: F) {
            let (l, r) = self.get_bounds(range);
            if l >= r {
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
                    self.all_apply(l, map);
                    l += 1;
                }
                if r & 1 == 1 {
                    r -= 1;
                    self.all_apply(r, map);
                }
                l >>= 1;
                r >>= 1;
            }
            l = l2;
            r = r2;

            for i in 1..=self.log {
                if ((l >> i) << i) != l {
                    self.upd(l >> i);
                }
                if ((r >> i) << i) != r {
                    self.upd((r - 1) >> i);
                }
            }
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
        pub fn partition_point(&mut self, l: usize, pred: impl Fn(T) -> bool) -> usize {
            if !pred(self.e) {
                return l;
            }

            if l == self.n {
                // `pred(self.e)` has already been checked that it's `true`.
                return self.n;
            }

            let mut l = l + self.size;
            for i in (1..=self.log).rev() {
                self.push(l >> i);
            }
            let mut sm = self.e;

            loop {
                l >>= l.trailing_zeros();
                if !pred((self.op)(sm, self.data[l])) {
                    while l < self.size {
                        self.push(l);
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
        pub fn left_partition_point(&mut self, r: usize, pred: impl Fn(T) -> bool) -> usize {
            if !pred(self.e) {
                return r + 1;
            }

            if r == 0 {
                // `pred(self.e)` has already been checked that it's `true`.
                return 0;
            }

            let mut r = r + self.size;
            for i in (1..=self.log).rev() {
                self.push((r - 1) >> i);
            }

            let mut sm = self.e;
            loop {
                r -= 1;
                while r > 1 && r & 1 == 1 {
                    r >>= 1;
                }

                if !pred((self.op)(self.data[r], sm)) {
                    while r < self.size {
                        self.push(r);
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

Last modified on 231007.