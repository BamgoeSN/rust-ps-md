# Bostan-Mori
Reference - [Alin Bostan, Ryuhei Mori. A Simple and Fast Algorithm for Computing the N-th Term of a Linearly Recurrent Sequence. SOSA’21 (SIAM Symposium on Simplicity in Algorithms), Jan 2021, Alexandria, United States. ffhal-02917827v2f](https://hal.inria.fr/hal-02917827v2/document)

Given \\(k\\) initial values \\(c_0,\ c_1,\ \cdots,\ c_{k-1}\\) and a linear recurrence of length \\(k\\):
\\[ f_{n+k} = c_0f_n + c_1f_{n+1} + \cdots + c_{k-1}f_{n+k-1} \\]
`bostan_mori::<P>(c, f, n)` calculates \\(f_n \bmod P\\). Here, \\(P\\) can be any integer larger than \\(1\\).

The time complexity of this algorithm is \\(O(\mathrm{M}(k) \log n)\\) where \\(\mathrm{M}(k)\\) represents the time complexity for polynomial multiplication.
This depends on the implementation of `poly_mul`. In this snippet, naive polynomial multiplication is used, resulting in a time complexity of \\(O(k^2 \log n)\\).
By replacing it with an implementation using FFT, the time complexity can be improved to \\(O(k \log k \log n)\\).

## Example
This example calculates terms of \\(\{f_n\}\\) for \\(0 \le n \le 10\\) where \\(f_0 = 0\\), \\(f_1 = 1\\), and \\(f_{n+2} = 2f_n + f_{n+1}\\).

```rust
# fn main() {
for i in 0..=10 {
    println!("{}", bostan_mori::<12345>(&[2, 1], &[0, 1], i));
}
# }
# 
# /// Finds arr[n] where
# /// arr[n+d] = rec[0]arr[n] + rec[1]arr[n+1] + rec[2]arr[n+2] + rec[3]arr[n+3] + ... + rec[d-1]arr[n+d-1]
# /// under mod P where d=rec.len()=arr.len()
# fn bostan_mori<const P: u64>(rec: &[u64], vals: &[u64], mut n: u64) -> u64 {
#     if vals.len() as u64 > n {
#         return vals[n as usize];
#     }
# 
#     let x = (0..rec.len()).find(|&i| rec[i] != 0).unwrap_or(rec.len());
#     if x == rec.len() {
#         return 0;
#     }
# 
#     let vals: Vec<u64> = vals.iter().map(|&v| v % P).collect();
#     let rec: Vec<u64> = rec.iter().map(|&v| v % P).collect();
#     let mut q = vec![1];
#     q.extend(rec.iter().rev().map(|&v| (P - v) % P));
#     let mut p = poly_mul::<P>(&vals, &q);
#     p.truncate(rec.len());
# 
#     while n >= 1 {
#         let mq: Vec<_> = q
#             .iter()
#             .enumerate()
#             .map(|(i, &v)| if i % 2 == 0 { v } else { (P - v) % P })
#             .collect();
#         let u = poly_mul::<P>(&p, &mq);
#         p = u
#             .iter()
#             .copied()
#             .skip((n % 2) as usize)
#             .step_by(2)
#             .collect();
#         let a = poly_mul::<P>(&q, &mq);
#         q = a.iter().copied().step_by(2).collect();
#         n /= 2;
#     }
# 
#     p[0] % P
# }
# 
# fn poly_mul<const P: u64>(a: &[u64], b: &[u64]) -> Vec<u64> {
#     let mut ret = vec![0; a.len() + b.len() - 1];
#     for (i, &av) in a.iter().enumerate() {
#         for (j, &bv) in b.iter().enumerate() {
#             ret[i + j] = (ret[i + j] + av * bv) % P;
#         }
#     }
#     ret
# }
# 
```
## Snippet

```rust,noplayground
/// Finds arr[n] where
/// arr[n+d] = rec[0]arr[n] + rec[1]arr[n+1] + rec[2]arr[n+2] + rec[3]arr[n+3] + ... + rec[d-1]arr[n+d-1]
/// under mod P where d=rec.len()=arr.len()
fn bostan_mori<const P: u64>(rec: &[u64], vals: &[u64], mut n: u64) -> u64 {
	if vals.len() as u64 > n {
		return vals[n as usize];
	}

	let x = (0..rec.len()).find(|&i| rec[i] != 0).unwrap_or(rec.len());
	if x == rec.len() {
		return 0;
	}

	let vals: Vec<u64> = vals.iter().map(|&v| v % P).collect();
	let rec: Vec<u64> = rec.iter().map(|&v| v % P).collect();
	let mut q = vec![1];
	q.extend(rec.iter().rev().map(|&v| (P - v) % P));
	let mut p = poly_mul::<P>(&vals, &q);
	p.truncate(rec.len());

	while n >= 1 {
		let mq: Vec<_> = q.iter().enumerate().map(|(i, &v)| if i % 2 == 0 { v } else { (P - v) % P }).collect();
		let u = poly_mul::<P>(&p, &mq);
		p = u.iter().copied().skip((n % 2) as usize).step_by(2).collect();
		let a = poly_mul::<P>(&q, &mq);
		q = a.iter().copied().step_by(2).collect();
		n /= 2;
	}

	p[0] % P
}

/// Naive O(k^2) multiplication
fn poly_mul<const P: u64>(a: &[u64], b: &[u64]) -> Vec<u64> {
	let mut ret = vec![0; a.len() + b.len() - 1];
	for (i, &av) in a.iter().enumerate() {
		for (j, &bv) in b.iter().enumerate() {
			ret[i + j] = (ret[i + j] + av * bv) % P;
		}
	}
	ret
}
```