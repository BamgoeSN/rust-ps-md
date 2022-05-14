# Bit Array
```rust,noplayground
struct BitArray {
    bytes: Vec<u8>,
}

impl BitArray {
    #[inline(always)]
    fn new(size: usize) -> Self {
        Self {
            bytes: vec![0; (size >> 3) + 1],
        }
    }

    #[inline(always)]
    fn get(&self, idx: usize) -> bool {
        (self.bytes[idx >> 3] >> (idx & 0b111)) & 1 == 1
    }

    #[inline(always)]
    fn set_true(&mut self, idx: usize) {
        self.bytes[idx >> 3] |= 1 << (idx & 0b111);
    }

    #[inline(always)]
    fn set_false(&mut self, idx: usize) {
        self.bytes[idx >> 3] &= !(1 << (idx & 0b111));
    }

    #[inline(always)]
    fn set(&mut self, idx: usize, to: bool) {
        if to {
            self.set_true(idx);
        } else {
            self.set_false(idx);
        }
    }

    #[inline(always)]
    fn toggle(&mut self, idx: usize) {
        self.bytes[idx >> 3] ^= 1 << (idx & 0b111);
    }

    #[inline(always)]
    fn fill(&mut self, with: u8) {
        for v in self.bytes.iter_mut() {
            *v = with;
        }
    }

    #[inline(always)]
    fn fill_range(&mut self, chunk_start: usize, chunk_end: usize, with: u8) {
        for v in self.bytes[chunk_start..chunk_end].iter_mut() {
            *v = with;
        }
    }
}
```