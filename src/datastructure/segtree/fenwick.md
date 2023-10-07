# Fenwick Tree

Given an integer array \\(A\\) of length \\(n\\), a Fenwick tree processes the following queries in \\(O(\log{n})\\) time:
- Add a certain amount to an element
- Calculate the sum of the elements of an interval

A Fenwick tree uses half the memory of a segment tree, but the performance in terms of time is just about the same.

A type of elements of \\(A\\) must be a primitive signed integer type, such as `i32` and `i64`, and floats such as `f64`. Unsigned integer types like `u64` do not work. Specifically, the type should implement `From<i8>`.

## Example

```rust
# fn main() {
let mut fw: Fenwick<i32> = Fenwick::new(10);
for i in 0..10 {
    print!("{} ", fw.get(i));
}
println!(); // 0 0 0 0 0 0 0 0 0 0
fw.add(2, 10);
fw.add(5, 100);
fw.add(3, -1);
for i in 0..10 {
    print!("{} ", fw.get(i));
}
println!(); // 0 0 10 -1 0 100 0 0 0 0
println!("{}", fw.sum(3..8)); // 99
# }
# 
# struct Fenwick<T> {
#     n: usize,
#     data: Vec<T>,
# }
# 
# impl<T: Copy + From<i8> + std::ops::AddAssign + std::ops::Sub<Output = T>> Fenwick<T> {
#     fn new(n: usize) -> Self {
#         Self {
#             n,
#             data: vec![0.into(); n],
#         }
#     }
# 
#     fn add(&mut self, idx: usize, val: T) {
#         let mut idx = idx + 1;
#         while idx <= self.n {
#             self.data[idx - 1] += val;
#             idx += idx & (!idx + 1);
#         }
#     }
# 
#     fn get(&self, idx: usize) -> T {
#         self.sum(idx..=idx)
#     }
# 
#     fn sum(&self, range: impl std::ops::RangeBounds<usize>) -> T {
#         use std::ops::Bound::*;
#         let l = match range.start_bound() {
#             Included(&v) => v,
#             Excluded(&v) => v + 1,
#             Unbounded => 0,
#         };
#         let r = match range.end_bound() {
#             Included(&v) => v + 1,
#             Excluded(&v) => v,
#             Unbounded => self.n,
#         };
#         self.inner_sum(r) - self.inner_sum(l)
#     }
# 
#     fn inner_sum(&self, mut r: usize) -> T {
#         let mut s: T = 0.into();
#         while r > 0 {
#             s += self.data[r - 1];
#             r -= r & (!r + 1);
#         }
#         s
#     }
# }
```

## Code

```rust,noplayground
struct Fenwick<T> {
	n: usize,
	data: Vec<T>,
}

impl<T: Copy + From<i8> + std::ops::AddAssign + std::ops::Sub<Output = T>> Fenwick<T> {
	fn new(n: usize) -> Self {
		Self { n, data: vec![0.into(); n] }
	}

	fn add(&mut self, idx: usize, val: T) {
		let mut idx = idx + 1;
		while idx <= self.n {
			self.data[idx - 1] += val;
			idx += idx & (!idx + 1);
		}
	}

	fn get(&self, idx: usize) -> T {
		self.sum(idx..=idx)
	}

	fn sum(&self, range: impl std::ops::RangeBounds<usize>) -> T {
		use std::ops::Bound::*;
		let l = match range.start_bound() {
			Included(&v) => v,
			Excluded(&v) => v + 1,
			Unbounded => 0,
		};
		let r = match range.end_bound() {
			Included(&v) => v + 1,
			Excluded(&v) => v,
			Unbounded => self.n,
		};
		self.inner_sum(r) - self.inner_sum(l)
	}

	fn inner_sum(&self, mut r: usize) -> T {
		let mut s: T = 0.into();
		while r > 0 {
			s += self.data[r - 1];
			r -= r & (!r + 1);
		}
		s
	}
}
```
