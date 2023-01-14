# Macros

## HashMap
```rust,noplayground
macro_rules! count_tts {
    () => { 0 };
    ($odd:tt $($a:tt $b:tt)*) => { (count_tts!($($a)*) << 1) | 1 };
    ($($a:tt $even:tt)*) => { count_tts!($($a)*) << 1 };
}

// let map: HashMap<i64, i64> = hashmap![1,1; 2,2; 3,3];
macro_rules! hashmap {
    ($($k:expr,$v:expr);*) => {{
        let mut map = HashMap::with_capacity(count_tts![$($k )*]);
        $( map.insert($k, $v); )*
        map
    }}
}
```

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
