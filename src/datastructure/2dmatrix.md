# 2D Matrix
So far, only the very basic functionalities are implemented.

## Code
```rust,noplayground
pub struct Mat<T> {
	row: usize,
	col: usize,
	data: Vec<T>,
}

impl<T> Mat<T> {
	pub fn new(row: usize, col: usize) -> Self
	where T: Default {
		Self { row, col, data: (0..row * col).map(|_| Default::default()).collect() }
	}

	pub fn resize_iter(row: usize, col: usize, data: impl IntoIterator<Item = T>) -> Self {
		let data: Vec<T> = data.into_iter().take(row * col).collect();
		debug_assert_eq!(data.len(), row * col);
		Self { row, col, data }
	}
}

impl<T> Index<usize> for Mat<T> {
	type Output = [T];
	fn index(&self, idx: usize) -> &Self::Output {
		let l = idx * self.col;
		&self.data[l..l + self.col]
	}
}

impl<T> IndexMut<usize> for Mat<T> {
	fn index_mut(&mut self, idx: usize) -> &mut Self::Output {
		let l = idx * self.col;
		&mut self.data[l..l + self.col]
	}
}
```
