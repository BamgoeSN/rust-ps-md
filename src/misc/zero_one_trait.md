# Zero/One Trait (Deprecated)

This trait was used for representing numeric values, but the author has learned that `Into` and `From` traits exist, hence this trait is now deprecated. Any other templates which includes or requires to include this trait will be fixed soon.

```rust,noplayground
pub trait ZeroOne: Sized + Copy {
    fn zero() -> Self;
    fn one() -> Self;
}

macro_rules! impl_zero_one {
    ($($ty:ty) *) => { $(
        impl ZeroOne for $ty {
            #[inline(always)]
            fn one() -> Self {1}
            #[inline(always)]
            fn zero() -> Self {0}
        }
    )+ };
}

impl_zero_one!(isize i8 i16 i32 i64 i128 usize u8 u16 u32 u64 u128);
```