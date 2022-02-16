# Zero/One Trait

## Zero
### Snippet
```rust,noplayground
pub trait Zero: Sized + Copy {
    fn zero() -> Self;
}

macro_rules! impl_zero {
    ($ty:ty) => {
        impl Zero for $ty {
            fn zero() -> Self {
                0
            }
        }
    };
}

impl_zero!(isize);
impl_zero!(i8);
impl_zero!(i16);
impl_zero!(i32);
impl_zero!(i64);
impl_zero!(i128);
impl_zero!(usize);
impl_zero!(u8);
impl_zero!(u16);
impl_zero!(u32);
impl_zero!(u64);
impl_zero!(u128);
```

## One
### Snippet
```rust, noplayground
pub trait One: Sized + Copy {
    fn one() -> Self;
}

macro_rules! impl_one {
    ($ty:ty) => {
        impl One for $ty {
            fn one() -> Self {
                1
            }
        }
    };
}

impl_one!(isize);
impl_one!(i8);
impl_one!(i16);
impl_one!(i32);
impl_one!(i64);
impl_one!(i128);
impl_one!(usize);
impl_one!(u8);
impl_one!(u16);
impl_one!(u32);
impl_one!(u64);
impl_one!(u128);
```