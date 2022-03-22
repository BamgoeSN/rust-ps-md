# Zero/One Trait

## Snippet
```rust,noplayground
pub trait ZeroOne: Sized + Copy {
    fn zero() -> Self;
    fn one() -> Self;
}

macro_rules! impl_zero_one {
    ($ty:ty) => {
        impl ZeroOne for $ty {
            #[inline(always)]
            fn one() -> Self {
                1
            }
            #[inline(always)]
            fn zero() -> Self {
                0
            }
        }
    };
}

impl_zero_one!(isize);
impl_zero_one!(i8);
impl_zero_one!(i16);
impl_zero_one!(i32);
impl_zero_one!(i64);
impl_zero_one!(i128);
impl_zero_one!(usize);
impl_zero_one!(u8);
impl_zero_one!(u16);
impl_zero_one!(u32);
impl_zero_one!(u64);
impl_zero_one!(u128);
```