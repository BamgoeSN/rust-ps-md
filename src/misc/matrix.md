# Fast n-dimensional Array

```rust,noplayground
struct Matrix<T>(Vec<T>, usize); // colnum

impl<T> std::ops::Index<usize> for Matrix<T> {
    type Output = [T];

    fn index(&self, i: usize) -> &Self::Output {
        &self.0[self.1 * i..self.1 * (i + 1)]
    }
}

impl<T> std::ops::IndexMut<usize> for Matrix<T> {
    fn index_mut(&mut self, i: usize) -> &mut Self::Output {
        &mut self.0[self.1 * i..self.1 * (i + 1)]
    }
}
```