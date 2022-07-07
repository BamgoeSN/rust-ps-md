# Manacher

For an array \\(A\\) of length \\(n\\), `manacher(A)` returns a vector \\(M\\) where, for every \\(i \in \left[0, n\right)\\), \\(A_{i-j} = A_{i+j}\\) holds for every \\(j \in \left[0, M_i \right)\\).

Additional modification should be added by a user to use this function for finding every palindromes among subsequences of a string.

## Example

```rust
# fn main() {
let s = "abracadacabra".as_bytes();
let man = manacher(s);
println!("{:?}", man); // [1, 1, 1, 1, 2, 1, 4, 1, 2, 1, 1, 1, 1]

for i in 0..s.len() {
    println!(
        "{}",
        std::str::from_utf8(&s[i + 1 - man[i]..i + man[i]]).unwrap()
    );
}
# }
#
# fn manacher<T: Eq>(arr: &[T]) -> Vec<usize> {
#     let n = arr.len();
#     let mut mana: Vec<usize> = vec![1; n];
#     let mut r: usize = 1;
#     let mut p: usize = 0;
# 
#     for i in 1..arr.len() {
#         if i + 1 >= r {
#             mana[i] = 1;
#         } else {
#             let j = 2 * p - i;
#             mana[i] = mana[j].min(r - i);
#         }
# 
#         while mana[i] <= i && i + mana[i] < n {
#             if arr[(i - mana[i])] != arr[(i + mana[i])] {
#                 break;
#             }
#             mana[i] += 1;
#         }
# 
#         if r < mana[i] + i {
#             r = mana[i] + i;
#             p = i;
#         }
#     }
# 
#     mana
# }
```

## Code

```rust,noplayground
fn manacher<T: Eq>(arr: &[T]) -> Vec<u32> {
    let n = arr.len() as u32;
    let mut mana: Vec<u32> = vec![1; n as usize];
    let mut r: u32 = 1;
    let mut p: u32 = 0;

    for i in 1..arr.len() as u32 {
        if i + 1 >= r {
            mana[i as usize] = 1;
        } else {
            let j = 2 * p - i;
            mana[i as usize] = mana[j as usize].min(r - i);
        }

        while mana[i as usize] <= i && i + mana[i as usize] < n {
            if arr[(i - mana[i as usize]) as usize] != arr[(i + mana[i as usize]) as usize] {
                break;
            }
            mana[i as usize] += 1;
        }

        if r < mana[i as usize] + i {
            r = mana[i as usize] + i;
            p = i;
        }
    }

    mana
}
```