# Number Theoretic Transform (NTT)

Number Theoretic Transform (NTT) is an alternative of FFT where the domain of numbers is \\(\mathbb{Z}_p\\) where \\(p\\) is a prime with a form of \\(p = a \times 2^b + 1\\).

`ntt::convolute(&[u64], &[u64]) -> Vec<u64>` convolutes two slices using two NTTs on two different primes and CRT, within a range that none of the numbers in the result exceed the range of `u64`.

## Example
```rust
# fn main() {
let a: Vec<u64> = vec![1, 2];
let b: Vec<u64> = vec![3, 4, 5];
println!("{:?}", ntt::convolute(&a, &b)); // [3, 10, 13, 10, 0, 0, 0, 0]
# }
# 
# mod ntt {
#     // FFT_constname convention following https://algoshitpo.github.io/2020/05/20/fft-ntt/
#     // p: prime for modulo
#     // w: primitive root of p
#     // p = a * 2^b + 1
# 
#     //             p  ntt_a ntt_b   ntt_w
#     //   998,244,353    119    23       3
#     // 2,281,701,377    17     27       3
#     // 2,483,027,969    37     26       3
#     // 2,113,929,217    63     25       5
#     //   104,857,601    25     22       3
#     // 1,092,616,193    521    21       3
# 
#     fn ceil_pow2(n: usize) -> u32 {
#         n.next_power_of_two().trailing_zeros()
#     }
# 
#     /// Reverses k trailing bits of n. Assumes that the rest of usize::BITS-k bits are all zero.
#     const fn reverse_trailing_bits(n: usize, k: u32) -> usize {
#         n.reverse_bits() >> (usize::BITS - k)
#     }
# 
#     #[derive(Clone, Debug)]
#     pub struct Ntt<const P: u64> {
#         pub arr: Vec<u64>,
#     }
# 
#     impl<const P: u64> Ntt<P> {
#         pub const fn ntt_a() -> u64 {
#             let mut p = P - 1;
#             while p & 1 == 0 {
#                 p >>= 1;
#             }
#             p
#         }
# 
#         pub const fn ntt_b() -> u32 {
#             let mut p = P - 1;
#             let mut ret = 0;
#             while p & 1 == 0 {
#                 p >>= 1;
#                 ret += 1;
#             }
#             ret
#         }
# 
#         pub const fn ntt_w() -> u64 {
#             match P {
#                 998244353 | 2281701377 | 2483027969 | 104857601 | 1092616193 => 3,
#                 2113929217 => 5,
#                 _ => todo!(),
#             }
#         }
# 
#         const fn pow(base: u64, exp: u64) -> u64 {
#             let mut base = base;
#             let mut exp = exp;
#             let mut ret = 1;
#             while exp != 0 {
#                 if exp & 1 != 0 {
#                     ret = ret * base % P;
#                 }
#                 base = base * base % P;
#                 exp >>= 1;
#             }
#             ret
#         }
# 
#         // unity(n, 1) ^ (1<<n) == 1
#         const fn unity(n: u32, k: u64) -> u64 {
#             Self::pow(Self::pow(Self::ntt_w(), Self::ntt_a()), k << (Self::ntt_b() - n))
#         }
# 
#         const fn recip(x: u64) -> u64 {
#             Self::pow(x, P - 2)
#         }
# 
#         pub fn new(arr: Vec<u64>) -> Self {
#             Self { arr }
#         }
# 
#         pub fn ntt(&mut self) {
#             let n: usize = self.arr.len();
#             let k = n.trailing_zeros();
#             debug_assert_eq!(n, 1 << k);
# 
#             for i in 0..n {
#                 let j = reverse_trailing_bits(i, k);
#                 if i < j {
#                     self.arr.swap(i, j);
#                 }
#             }
# 
#             for x in 0..k {
#                 let base: u64 = Self::unity(x + 1, 1);
#                 let s = 1 << x;
#                 for i in (0..n).step_by(s << 1) {
#                     let mut mult: u64 = 1;
#                     for j in 0..s {
#                         let tmp = (self.arr[i + j + s] * mult) % P;
#                         self.arr[i + j + s] = (self.arr[i + j] + P - tmp) % P;
#                         self.arr[i + j] = (self.arr[i + j] + tmp) % P;
#                         mult = mult * base % P;
#                     }
#                 }
#             }
#         }
# 
#         pub fn intt(&mut self) {
#             let n: usize = self.arr.len();
#             let k = n.trailing_zeros();
#             debug_assert_eq!(n, 1 << k);
# 
#             for i in 0..n {
#                 let j = reverse_trailing_bits(i, k);
#                 if i < j {
#                     self.arr.swap(i, j);
#                 }
#             }
# 
#             for x in 0..k {
#                 let base: u64 = Self::recip(Self::unity(x + 1, 1));
#                 let s = 1 << x;
#                 for i in (0..n).step_by(s << 1) {
#                     let mut mult: u64 = 1;
#                     for j in 0..s {
#                         let tmp = (self.arr[i + j + s] * mult) % P;
#                         self.arr[i + j + s] = (self.arr[i + j] + P - tmp) % P;
#                         self.arr[i + j] = (self.arr[i + j] + tmp) % P;
#                         mult = mult * base % P;
#                     }
#                 }
#             }
# 
#             let r = Self::recip(n as u64);
#             for f in self.arr.iter_mut() {
#                 *f *= r;
#                 *f %= P;
#             }
#         }
# 
#         pub fn convolute(a: &[u64], b: &[u64]) -> Self {
#             let nlen = 1 << ceil_pow2(a.len() + b.len());
#             let pad = |a: &[u64]| a.iter().copied().chain(std::iter::repeat(0)).take(nlen).collect();
#             let arr = pad(a);
#             let brr = pad(b);
# 
#             let mut arr = Self::new(arr);
#             let mut brr = Self::new(brr);
#             arr.ntt();
#             brr.ntt();
# 
#             let crr: Vec<_> = arr.arr.iter().zip(brr.arr.iter()).map(|(&a, &b)| a * b % P).collect();
#             let mut crr = Self::new(crr);
#             crr.intt();
#             crr
#         }
#     }
# 
#     fn merge<const P: u64, const Q: u64>(one: &[u64], two: &[u64]) -> Vec<u64> {
#         let p: u64 = Ntt::<Q>::recip(P);
#         let q: u64 = Ntt::<P>::recip(Q);
#         let r: u64 = P * Q;
#         one.iter()
#             .zip(two.iter())
#             .map(|(&a1, &a2)| ((a1 as u128 * q as u128 * Q as u128 + a2 as u128 * p as u128 * P as u128) % r as u128) as u64)
#             .collect()
#     }
# 
#     pub fn convolute(a: &[u64], b: &[u64]) -> Vec<u64> {
#         const P: u64 = 2281701377;
#         const Q: u64 = 998244353;
#         let a: Vec<u64> = a.iter().copied().collect();
#         let b: Vec<u64> = b.iter().copied().collect();
# 
#         let arr = Ntt::<P>::convolute(&a, &b);
#         let brr = Ntt::<Q>::convolute(&a, &b);
#         merge::<P, Q>(&arr.arr, &brr.arr)
#     }
# }
```

## Code

```rust,noplayground
mod ntt {
	// FFT_constname convention following https://algoshitpo.github.io/2020/05/20/fft-ntt/
	// p: prime for modulo
	// w: primitive root of p
	// p = a * 2^b + 1

	//             p  ntt_a ntt_b   ntt_w
	//   998,244,353    119    23       3
	// 2,281,701,377    17     27       3
	// 2,483,027,969    37     26       3
	// 2,113,929,217    63     25       5
	//   104,857,601    25     22       3
	// 1,092,616,193    521    21       3

	fn ceil_pow2(n: usize) -> u32 { n.next_power_of_two().trailing_zeros() }

	/// Reverses k trailing bits of n. Assumes that the rest of usize::BITS-k bits are all zero.
	const fn reverse_trailing_bits(n: usize, k: u32) -> usize { n.reverse_bits() >> (usize::BITS - k) }

	#[derive(Clone, Debug)]
	pub struct Ntt<const P: u64> {
		pub arr: Vec<u64>,
	}

	impl<const P: u64> Ntt<P> {
		pub const fn ntt_a() -> u64 {
			let p = P - 1;
			p >> p.trailing_zeros()
		}

		pub const fn ntt_b() -> u32 { (P - 1).trailing_zeros() }

		/// Primitive root of P
		pub const fn ntt_w() -> u64 {
			match P {
				998244353 | 2281701377 | 2483027969 | 104857601 | 1092616193 => 3,
				2113929217 => 5,
				_ => todo!(),
			}
		}

		const fn pow(mut base: u64, mut exp: u64) -> u64 {
			let mut ret = 1;
			while exp != 0 {
				if exp & 1 != 0 {
					ret = ret * base % P;
				}
				base = base * base % P;
				exp >>= 1;
			}
			ret
		}

		/// Returns an integer x where x^(2^n) == 1 mod P.
		/// That is, it returns (2^n)-th root of unity.
		const fn unity(n: u32, k: u64) -> u64 { Self::pow(Self::pow(Self::ntt_w(), Self::ntt_a()), k << (Self::ntt_b() - n)) }

		const fn recip(x: u64) -> u64 { Self::pow(x, P - 2) }

		pub fn new(arr: Vec<u64>) -> Self { Self { arr } }

		pub fn ntt(&mut self) {
			let n: usize = self.arr.len();
			let k = n.trailing_zeros();
			debug_assert_eq!(n, 1 << k);

			for i in 0..n {
				let j = reverse_trailing_bits(i, k);
				if i < j {
					self.arr.swap(i, j);
				}
			}

			let mut basis = vec![Self::unity(k, 1)];
			for i in 1..k as usize {
				basis.push(basis[i - 1] * basis[i - 1] % P);
			}
			for (x, &base) in basis.iter().rev().enumerate() {
				let s = 1 << x;
				for i in (0..n).step_by(s << 1) {
					let mut mult: u64 = 1;
					for j in i..i + s {
						let tmp = (self.arr[j + s] * mult) % P;
						self.arr[j + s] = (self.arr[j] + P - tmp) % P;
						self.arr[j] = (self.arr[j] + tmp) % P;
						mult = mult * base % P;
					}
				}
			}
		}

		pub fn intt(&mut self) {
			let n: usize = self.arr.len();
			let k = n.trailing_zeros();
			debug_assert_eq!(n, 1 << k);

			for i in 0..n {
				let j = reverse_trailing_bits(i, k);
				if i < j {
					self.arr.swap(i, j);
				}
			}

			let mut basis = vec![Self::recip(Self::unity(k, 1))];
			for i in 1..k as usize {
				basis.push(basis[i - 1] * basis[i - 1] % P);
			}
			for (x, &base) in basis.iter().rev().enumerate() {
				let s = 1 << x;
				for i in (0..n).step_by(s << 1) {
					let mut mult: u64 = 1;
					for j in i..i + s {
						let tmp = (self.arr[j + s] * mult) % P;
						self.arr[j + s] = (self.arr[j] + P - tmp) % P;
						self.arr[j] = (self.arr[j] + tmp) % P;
						mult = mult * base % P;
					}
				}
			}

			let r = Self::recip(n as u64);
			for f in self.arr.iter_mut() {
				*f *= r;
				*f %= P;
			}
		}

		pub fn convolute(a: &[u64], b: &[u64]) -> Self {
			let nlen = 1 << ceil_pow2(a.len() + b.len());
			let pad = |a: &[u64]| a.iter().copied().chain(std::iter::repeat(0)).take(nlen).collect();
			let arr = pad(a);
			let brr = pad(b);

			let mut arr = Self::new(arr);
			let mut brr = Self::new(brr);
			arr.ntt();
			brr.ntt();

			let crr: Vec<_> = arr.arr.iter().zip(brr.arr.iter()).map(|(&a, &b)| a * b % P).collect();
			let mut crr = Self::new(crr);
			crr.intt();
			crr
		}
	}

	fn merge<const P: u64, const Q: u64>(one: &[u64], two: &[u64]) -> Vec<u64> {
		let p = Ntt::<Q>::recip(P) as u128;
		let q = Ntt::<P>::recip(Q) as u128;
		let [pp, qq] = [P, Q].map(|x| x as u128);
		let r = (P * Q) as u128;

		one.iter()
			.zip(two.iter())
			.map(|(&a1, &a2)| {
				let [a, b] = [a1, a2].map(|x| x as u128);
				(a * q * qq + b * p * pp) % r
			})
			.map(|x| x as u64)
			.collect()
	}

	pub fn convolute(a: &[u64], b: &[u64]) -> Vec<u64> {
		const P: u64 = 2281701377;
		const Q: u64 = 998244353;

		let arr = Ntt::<P>::convolute(a, b);
		let brr = Ntt::<Q>::convolute(a, b);
		merge::<P, Q>(&arr.arr, &brr.arr)
	}
}
```

---

Last modified on 231203.