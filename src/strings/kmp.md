# KMP

Given an array \\(P\\), `KMPNeedle::new(P)` returns a `KMPNeedle` instance for searching \\(P\\) in any array, using `KMPMatcher`.

Given a `KMPNeedle` instance `p` and an array target \\(T\\), `KMPMatcher::new(&p, T)` returns an iterator yielding indicies where \\(P\\) appears within \\(T\\). Even if some ranges where \\(P\\) exists within \\(T\\) overlap with each other, they are still all yielded as shown in the second string from the example `"ABCDABCDABCD"`.

`KMPNeedle::new(P)` runs in a time complexity of \\(O(\left| P \right|)\\) and `KMPMatcher::new(T)` takes a time complexity of \\(O(\left| T \right|)\\) to be consumed. `KMPMatcher` searches for a pattern lazily, so it only takes time of total searched length in \\(T\\).

A single `KMPNeedle` instance can be used to search \\(P\\) in multiple arrays as shown in the example.

## Example

```rust
# fn main() {
let pattern = "ABCDABC";
let targets = vec!["ABDABCDABCE", "ABCDABCDABCD", "ABBCCABCDABDABCDABC"];

let needle = KMPNeedle::new(pattern.as_bytes());
for &t in targets.iter() {
    let kmp = KMPMatcher::new(&needle, t.as_bytes());
    for v in kmp {
        print!("{} ", v);
    }
    println!();
}
# }
# 
# struct KMPNeedle<'a, T: PartialEq> {
#     p: &'a [T],
#     c: Vec<usize>,
# }
# 
# impl<'a, T: PartialEq> KMPNeedle<'a, T> {
#     fn new(p: &'a [T]) -> Self {
#         let mut c: Vec<usize> = vec![0; p.len() + 1];
# 
#         let mut l = 0;
#         for (r, v) in p.iter().enumerate().skip(1) {
#             while l > 0 && p[l] != *v {
#                 l = c[l];
#             }
#             if p[l] == *v {
#                 c[r + 1] = l + 1;
#                 l += 1;
#             }
#         }
# 
#         Self { p, c }
#     }
# }
# 
# struct KMPMatcher<'a, 'b: 'a, 'c: 'b, T: PartialEq> {
#     needle: &'c KMPNeedle<'b, T>,
#     t: &'a [T],
#     i: usize,
#     j: usize,
# }
# 
# impl<'a, 'b: 'a, 'c: 'b, T: PartialEq> KMPMatcher<'a, 'b, 'c, T> {
#     fn new(needle: &'c KMPNeedle<'b, T>, t: &'a [T]) -> Self {
#         Self {
#             needle,
#             t,
#             i: 0,
#             j: 0,
#         }
#     }
# }
# 
# impl<'a, 'b: 'a, 'c: 'b, T: PartialEq> Iterator for KMPMatcher<'a, 'b, 'c, T> {
#     type Item = usize;
# 
#     fn next(&mut self) -> Option<Self::Item> {
#         while self.i < self.t.len() {
#             while self.j > 0 && self.t[self.i] != self.needle.p[self.j] {
#                 self.j = self.needle.c[self.j];
#             }
#             if self.t[self.i] == self.needle.p[self.j] {
#                 if self.j == self.needle.p.len() - 1 {
#                     self.j = self.needle.c[self.j + 1];
#                     self.i += 1;
#                     return Some(self.i - self.needle.p.len());
#                 } else {
#                     self.j += 1;
#                 }
#             }
#             self.i += 1;
#         }
#         None
#     }
# }
```

## Code

```rust,noplayground
struct KMPNeedle<'a, T: PartialEq> {
    p: &'a [T],
    c: Vec<usize>,
}

impl<'a, T: PartialEq> KMPNeedle<'a, T> {
    fn new(p: &'a [T]) -> Self {
        let mut c: Vec<usize> = vec![0; p.len() + 1];

        let mut l = 0;
        for (r, v) in p.iter().enumerate().skip(1) {
            while l > 0 && p[l] != *v {
                l = c[l];
            }
            if p[l] == *v {
                c[r + 1] = l + 1;
                l += 1;
            }
        }

        Self { p, c }
    }
}

struct KMPMatcher<'a, 'b: 'a, 'c: 'b, T: PartialEq> {
    needle: &'c KMPNeedle<'b, T>,
    t: &'a [T],
    i: usize,
    j: usize,
}

impl<'a, 'b: 'a, 'c: 'b, T: PartialEq> KMPMatcher<'a, 'b, 'c, T> {
    fn new(needle: &'c KMPNeedle<'b, T>, t: &'a [T]) -> Self {
        Self {
            needle,
            t,
            i: 0,
            j: 0,
        }
    }
}

impl<'a, 'b: 'a, 'c: 'b, T: PartialEq> Iterator for KMPMatcher<'a, 'b, 'c, T> {
    type Item = usize;

    fn next(&mut self) -> Option<Self::Item> {
        while self.i < self.t.len() {
            while self.j > 0 && self.t[self.i] != self.needle.p[self.j] {
                self.j = self.needle.c[self.j];
            }
            if self.t[self.i] == self.needle.p[self.j] {
                if self.j == self.needle.p.len() - 1 {
                    self.j = self.needle.c[self.j + 1];
                    self.i += 1;
                    return Some(self.i - self.needle.p.len());
                } else {
                    self.j += 1;
                }
            }
            self.i += 1;
        }
        None
    }
}
```