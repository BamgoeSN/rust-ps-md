# KMP

Given an array `pattern`, `failure_function(pattern)` returns a failure function `failure` of `pattern`.

Given an array `haystack`, `kmp_search(haystack, pattern, failure)` returns a result of searching `pattern` in `haystack`, given that `failure` is a proper failure function of `pattern`. Denoting the returned array as `result` and the length of `pattern` as `n`, if `result[i + n] == n`, then `result[i..n] == pattern`.

The API looks like this, as the failure function itself is needed in many algorithm problems, rather than directly using KMP for a string searching.

## Example

```rust
# fn main() {
let pattern = b"ABCDABC";
let targets = ["ABDABCDABCE", "ABCDABCDABCD", "ABBCCABCDABDABCDABC"].map(|b| b.as_bytes());

let failure = failure_function(pattern);
for &t in &targets {
    let result = kmp_search(t, pattern, &failure);
    println!("{:?}", result);
    for i in 0..result.len() - pattern.len() {
        if result[i + pattern.len()] == pattern.len() {
            print!("{} ", i);
        }
    }
    println!();
}
# }
# 
# /// Returns a failure function of `pattern`.
# fn failure_function<T: PartialEq>(pattern: &[T]) -> Vec<usize> {
#     let n = pattern.len();
#     let mut c = vec![0, 0];
#     let mut x;
#     for i in 1..n {
#         x = c[i];
#         loop {
#             if pattern[i] == pattern[x] {
#                 c.push(x + 1);
#                 break;
#             }
#             if x == 0 {
#                 c.push(0);
#                 break;
#             }
#             x = c[x];
#         }
#     }
#     c
# }
# 
# /// Returns a result of KMP search.
# /// For `n = pattern.len()`, if `result[i] == n`, then `haystack[i-n..i] == pattern`.
# fn kmp_search<T: PartialEq>(haystack: &[T], pattern: &[T], failure: &[usize]) -> Vec<usize> {
#     let m = haystack.len();
#     let mut d = vec![0];
#     let mut x;
#     for i in 0..m {
#         x = d[i];
#         if x == pattern.len() {
#             x = failure[x];
#         }
#         loop {
#             if haystack[i] == pattern[x] {
#                 d.push(x + 1);
#                 break;
#             }
#             if x == 0 {
#                 d.push(0);
#                 break;
#             }
#             x = failure[x];
#         }
#     }
#     d
# }
```

## Code

```rust,noplayground
/// Returns a failure function of `pattern`.
fn failure_function<T: PartialEq>(pattern: &[T]) -> Vec<usize> {
    let n = pattern.len();
    let mut c = vec![0, 0];
    let mut x;
    for i in 1..n {
        x = c[i];
        loop {
            if pattern[i] == pattern[x] {
                c.push(x + 1);
                break;
            }
            if x == 0 {
                c.push(0);
                break;
            }
            x = c[x];
        }
    }
    c
}

/// Returns a result of KMP search.
/// For `n = pattern.len()`, if `result[i] == n`, then `haystack[i-n..i] == pattern`.
fn kmp_search<T: PartialEq>(haystack: &[T], pattern: &[T], failure: &[usize]) -> Vec<usize> {
    let m = haystack.len();
    let mut d = vec![0];
    let mut x;
    for i in 0..m {
        x = d[i];
        if x == pattern.len() {
            x = failure[x];
        }
        loop {
            if haystack[i] == pattern[x] {
                d.push(x + 1);
                break;
            }
            if x == 0 {
                d.push(0);
                break;
            }
            x = failure[x];
        }
    }
    d
}
```

---

Last modified on 231008.