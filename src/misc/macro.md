# Macros

## Conversion between Enum and `u32`

Macro from <https://stackoverflow.com/questions/28028854/how-do-i-match-enum-values-with-an-integer>.

### Example
```rust
# use std::convert::{TryFrom, TryInto};
# macro_rules! back_to_enum {
#     {$(#[$meta:meta])* $vis:vis enum $name:ident {
#         $($(#[$vmeta:meta])* $vname:ident $(= $val:expr)?,)*
#     }} => {
#         $(#[$meta])*
#         $vis enum $name {
#             $($(#[$vmeta])* $vname $(= $val)?,)*
#         }
# 
#         impl TryFrom<u32> for $name {
#             type Error = ();
# 
#             fn try_from(v: u32) -> Result<Self, Self::Error> {
#                 match v {
#                     $(x if x == $name::$vname as u32 => Ok($name::$vname),)*
#                     _ => Err(()),
#                 }
#             }
#         }
#     }
# }
# 
back_to_enum! {
    #[derive(Clone, Copy, Debug)]
    enum Number {
        Zero,
        One,
        Two,
    }
}

# fn main() {
let i: u32 = 1;
let n: Number = i.try_into().unwrap();
println!("{:?}", n);  // One
let i = n as i32;
println!("{}", i);    // 1
# }
```

### Code
```rust,noplayground
use std::convert::{TryFrom, TryInto};
macro_rules! back_to_enum {
    {$(#[$meta:meta])* $vis:vis enum $name:ident {
        $($(#[$vmeta:meta])* $vname:ident $(= $val:expr)?,)*
    }} => {
        $(#[$meta])*
        $vis enum $name {
            $($(#[$vmeta])* $vname $(= $val)?,)*
        }
        impl TryFrom<u32> for $name {
            type Error = ();
            fn try_from(v: u32) -> Result<Self, Self::Error> {
                match v {
                    $(x if x == $name::$vname as u32 => Ok($name::$vname),)*
                    _ => Err(()),
                }
            }
        }
    }
}
```

---

Last modified on 231008.