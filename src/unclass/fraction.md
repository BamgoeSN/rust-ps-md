# Fraction

TODO: description

## Example
```rust
# fn main() {
use frac::*;
let mut a = Frac::new(8374927, 2983178);
println!("{a}");

let b: i64 = 31;
a += b;
println!("{a}");

let af: f64 = a.into();
println!("{af:.5}");

let (l, r) = a.lower_den(100);
println!("{l} <= {a} <= {r}");
let (lf, rf): (f64, f64) = (l.into(), r.into());
println!("{lf:.5} {rf:.5}");
# }
# 
# mod frac {
#     use std::{fmt::Display, ops::*};
# 
#     type I = i64;
#     type U = u64;
# 
#     #[derive(Clone, Copy, Debug)]
#     pub struct Frac {
#         pub num: I,
#         pub den: U,
#     }
# 
#     impl Frac {
#         pub fn simplify(self) -> Self {
#             fn gcd(mut a: U, mut b: U) -> U {
#                 while b != 0 {
#                     (a, b) = (b, a % b);
#                 }
#                 a
#             }
#             let g = gcd(self.num.abs() as U, self.den);
#             Self {
#                 num: self.num / g as I,
#                 den: self.den / g,
#             }
#         }
# 
#         pub fn new(num: I, den: U) -> Self {
#             assert_ne!(den, 0);
#             Self { num, den }.simplify()
#         }
# 
#         pub fn recip(self) -> Self {
#             if self.num < 0 {
#                 Self {
#                     num: -(self.den as I),
#                     den: (-self.num) as U,
#                 }
#             } else {
#                 Self {
#                     num: self.den as I,
#                     den: self.num as U,
#                 }
#             }
#         }
# 
#         pub fn floor(self) -> I {
#             let Self { num, den } = self;
#             let den = den as I;
#             num.div_euclid(den)
#         }
# 
#         pub fn ceil(self) -> I {
#             let Self { num, den } = self;
#             let den = den as I;
#             (num + den - 1).div_euclid(den)
#         }
# 
#         pub fn round(self) -> I {
#             let Self { num, den } = self;
#             let den = den as I;
#             (2 * num + den).div_euclid(2 * den)
#         }
# 
#         pub fn fract(self) -> Self {
#             self - self.floor()
#         }
# 
#         pub fn wrap(x: f64, max_den: U) -> (Self, Self) {
#             let ipart = x.floor() as I;
#             let d = x.fract();
#             if d == 0. {
#                 return (ipart.into(), ipart.into());
#             }
# 
#             let [(mut ln, mut ld), (mut rn, mut rd)]: [(U, U); 2] = [(0, 1), (1, 1)];
#             while (ln, ld) != (rn, rd) {
#                 let (pl, pr) = ((ln, ld), (rn, rd));
# 
#                 // Update l
#                 let k1 = (ld as f64 * d - ln as f64).div_euclid(rn as f64 - rd as f64 * d) as U;
#                 let k2 = (max_den - ld).div_euclid(rd);
#                 let k = k1.min(k2);
#                 ln += k * rn;
#                 ld += k * rd;
# 
#                 // Update r
#                 let k1 = (rn as f64 - rd as f64 * d).div_euclid(ld as f64 * d - ln as f64) as U;
#                 let k2 = (max_den - rd).div_euclid(ld);
#                 let k = k1.min(k2);
#                 rn += k * ln;
#                 rd += k * ld;
# 
#                 if pl == (ln, ld) && pr == (rn, rd) {
#                     break;
#                 }
#             }
# 
#             let l = Self::new(ln as I, ld) + ipart;
#             let r = Self::new(rn as I, rd) + ipart;
#             if x == l.into() {
#                 (l, l)
#             } else if x == r.into() {
#                 (r, r)
#             } else {
#                 (l, r)
#             }
#         }
# 
#         pub fn lower_den(self, max_den: U) -> (Self, Self) {
#             if self.den <= max_den {
#                 return (self, self);
#             }
# 
#             let ipart = self.floor();
#             let Self { num: dn, den: dd } = self.fract();
#             let dn = dn as U;
# 
#             let [(mut ln, mut ld), (mut rn, mut rd)]: [(U, U); 2] = [(0, 1), (1, 1)];
#             while (ln, ld) != (rn, rd) {
#                 let (pl, pr) = ((ln, ld), (rn, rd));
# 
#                 // Update l
#                 let k1 = (ld * dn - ln * dd).div_euclid(rn * dd - rd * dn);
#                 let k2 = (max_den - ld).div_euclid(rd);
#                 let k = k1.min(k2);
#                 ln += k * rn;
#                 ld += k * rd;
# 
#                 // Update r
#                 let k1 = (rn * dd - rd * dn).div_euclid(ld * dn - ln * dd);
#                 let k2 = (max_den - rd).div_euclid(ld);
#                 let k = k1.min(k2);
#                 rn += k * ln;
#                 rd += k * ld;
# 
#                 if pl == (ln, ld) && pr == (rn, rd) {
#                     break;
#                 }
#             }
# 
#             let l = Self::new(ln as I, ld) + ipart;
#             let r = Self::new(rn as I, rd) + ipart;
#             (l, r)
#         }
#     }
# 
#     impl Default for Frac {
#         fn default() -> Self {
#             Frac { num: 0, den: 1 }
#         }
#     }
# 
#     impl Display for Frac {
#         fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
#             write!(f, "{}/{}", self.num, self.den)
#         }
#     }
# 
#     impl From<I> for Frac {
#         fn from(num: I) -> Self {
#             Self { num, den: 1 }
#         }
#     }
# 
#     impl From<Frac> for f64 {
#         fn from(value: Frac) -> Self {
#             value.num as f64 / value.den as f64
#         }
#     }
# 
#     impl Neg for Frac {
#         type Output = Self;
#         fn neg(self) -> Self::Output {
#             Self {
#                 num: -self.num,
#                 den: self.den,
#             }
#         }
#     }
# 
#     impl Add for Frac {
#         type Output = Self;
#         fn add(self, rhs: Self) -> Self::Output {
#             Self {
#                 num: self.num * rhs.den as I + self.den as I * rhs.num,
#                 den: self.den * rhs.den,
#             }
#             .simplify()
#         }
#     }
# 
#     impl Sub for Frac {
#         type Output = Self;
#         fn sub(self, rhs: Self) -> Self::Output {
#             Self {
#                 num: self.num * rhs.den as I - self.den as I * rhs.num,
#                 den: self.den * rhs.den,
#             }
#             .simplify()
#         }
#     }
# 
#     impl Mul for Frac {
#         type Output = Self;
#         fn mul(self, rhs: Self) -> Self::Output {
#             Self {
#                 num: self.num * rhs.num,
#                 den: self.den * rhs.den,
#             }
#             .simplify()
#         }
#     }
# 
#     impl Div for Frac {
#         type Output = Self;
#         fn div(self, rhs: Self) -> Self::Output {
#             self * rhs.recip()
#         }
#     }
# 
#     impl PartialEq for Frac {
#         fn eq(&self, rhs: &Self) -> bool {
#             (self.num * rhs.den as I).eq(&(rhs.num * self.den as I))
#         }
#     }
# 
#     impl Eq for Frac {}
# 
#     impl PartialOrd for Frac {
#         fn partial_cmp(&self, rhs: &Self) -> Option<std::cmp::Ordering> {
#             (self.num * rhs.den as I).partial_cmp(&(rhs.num * self.den as I))
#         }
#     }
# 
#     impl Ord for Frac {
#         fn cmp(&self, rhs: &Self) -> std::cmp::Ordering {
#             (self.num * rhs.den as I).cmp(&(rhs.num * self.den as I))
#         }
#     }
# 
#     impl Add<I> for Frac {
#         type Output = Self;
#         fn add(self, rhs: I) -> Self::Output {
#             let rhs: Frac = rhs.into();
#             self + rhs
#         }
#     }
# 
#     impl Sub<I> for Frac {
#         type Output = Self;
#         fn sub(self, rhs: I) -> Self::Output {
#             let rhs: Frac = rhs.into();
#             self - rhs
#         }
#     }
# 
#     impl Mul<I> for Frac {
#         type Output = Self;
#         fn mul(self, rhs: I) -> Self::Output {
#             let rhs: Frac = rhs.into();
#             self * rhs
#         }
#     }
# 
#     impl Div<I> for Frac {
#         type Output = Self;
#         fn div(self, rhs: I) -> Self::Output {
#             let rhs: Frac = rhs.into();
#             self / rhs
#         }
#     }
# 
#     impl Add<Frac> for I {
#         type Output = Frac;
#         fn add(self, rhs: Frac) -> Self::Output {
#             rhs + self
#         }
#     }
# 
#     impl Sub<Frac> for I {
#         type Output = Frac;
#         fn sub(self, rhs: Frac) -> Self::Output {
#             -rhs + self
#         }
#     }
# 
#     impl Mul<Frac> for I {
#         type Output = Frac;
#         fn mul(self, rhs: Frac) -> Self::Output {
#             rhs * self
#         }
#     }
# 
#     impl Div<Frac> for I {
#         type Output = Frac;
#         fn div(self, rhs: Frac) -> Self::Output {
#             let lhs: Frac = self.into();
#             lhs / rhs
#         }
#     }
# 
#     impl AddAssign for Frac {
#         fn add_assign(&mut self, rhs: Self) {
#             *self = *self + rhs;
#         }
#     }
# 
#     impl SubAssign for Frac {
#         fn sub_assign(&mut self, rhs: Self) {
#             *self = *self - rhs;
#         }
#     }
# 
#     impl MulAssign for Frac {
#         fn mul_assign(&mut self, rhs: Self) {
#             *self = *self * rhs;
#         }
#     }
# 
#     impl DivAssign for Frac {
#         fn div_assign(&mut self, rhs: Self) {
#             *self = *self / rhs;
#         }
#     }
# 
#     impl AddAssign<I> for Frac {
#         fn add_assign(&mut self, rhs: I) {
#             *self = *self + rhs;
#         }
#     }
# 
#     impl SubAssign<I> for Frac {
#         fn sub_assign(&mut self, rhs: I) {
#             *self = *self - rhs;
#         }
#     }
# 
#     impl MulAssign<I> for Frac {
#         fn mul_assign(&mut self, rhs: I) {
#             *self = *self * rhs;
#         }
#     }
# 
#     impl DivAssign<I> for Frac {
#         fn div_assign(&mut self, rhs: I) {
#             *self = *self / rhs;
#         }
#     }
# }
```

## Code
```rust,noplayground
mod frac {
	use std::{fmt::Display, ops::*};

	type I = i64;
	type U = u64;

	#[derive(Clone, Copy, Debug)]
	pub struct Frac {
		pub num: I,
		pub den: U,
	}

	impl Frac {
		pub fn simplify(self) -> Self {
			fn gcd(mut a: U, mut b: U) -> U {
				while b != 0 {
					(a, b) = (b, a % b);
				}
				a
			}
			let g = gcd(self.num.abs() as U, self.den);
			Self {
				num: self.num / g as I,
				den: self.den / g,
			}
		}

		pub fn new(num: I, den: U) -> Self {
			assert_ne!(den, 0);
			Self { num, den }.simplify()
		}

		pub fn recip(self) -> Self {
			use std::cmp::Ordering::*;
			match self.num.cmp(&0) {
				Less => Self {		
					num: -(self.den as I),
					den: (-self.num) as U,
				},
				Equal => panic!("Reciprocal of zero"),
				Greater => Self {
					num: self.den as I,
					den: self.num as U,
				},
			}
		}

		pub fn floor(self) -> I {
			let Self { num, den } = self;
			let den = den as I;
			num.div_euclid(den)
		}

		pub fn ceil(self) -> I {
			let Self { num, den } = self;
			let den = den as I;
			(num + den - 1).div_euclid(den)
		}

		pub fn round(self) -> I {
			let Self { num, den } = self;
			let den = den as I;
			(2 * num + den).div_euclid(2 * den)
		}

		pub fn fract(self) -> Self {
			self - self.floor()
		}

		pub fn wrap(x: f64, max_den: U) -> (Self, Self) {
			let ipart = x.floor() as I;
			let d = x.fract();
			if d == 0. {
				return (ipart.into(), ipart.into());
			}

			let [(mut ln, mut ld), (mut rn, mut rd)]: [(U, U); 2] = [(0, 1), (1, 1)];
			while (ln, ld) != (rn, rd) {
				let (pl, pr) = ((ln, ld), (rn, rd));

				// Update l
				let k1 = (ld as f64 * d - ln as f64).div_euclid(rn as f64 - rd as f64 * d) as U;
				let k2 = (max_den - ld).div_euclid(rd);
				let k = k1.min(k2);
				ln += k * rn;
				ld += k * rd;

				// Update r
				let k1 = (rn as f64 - rd as f64 * d).div_euclid(ld as f64 * d - ln as f64) as U;
				let k2 = (max_den - rd).div_euclid(ld);
				let k = k1.min(k2);
				rn += k * ln;
				rd += k * ld;

				if pl == (ln, ld) && pr == (rn, rd) {
					break;
				}
			}

			let l = Self::new(ln as I, ld) + ipart;
			let r = Self::new(rn as I, rd) + ipart;
			if x == l.into() {
				(l, l)
			} else if x == r.into() {
				(r, r)
			} else {
				(l, r)
			}
		}

		pub fn lower_den(self, max_den: U) -> (Self, Self) {
			if self.den <= max_den {
				return (self, self);
			}

			let ipart = self.floor();
			let Self { num: dn, den: dd } = self.fract();
			let dn = dn as U;

			let [(mut ln, mut ld), (mut rn, mut rd)]: [(U, U); 2] = [(0, 1), (1, 1)];
			while (ln, ld) != (rn, rd) {
				let (pl, pr) = ((ln, ld), (rn, rd));

				// Update l
				let k1 = (ld * dn - ln * dd).div_euclid(rn * dd - rd * dn);
				let k2 = (max_den - ld).div_euclid(rd);
				let k = k1.min(k2);
				ln += k * rn;
				ld += k * rd;

				// Update r
				let k1 = (rn * dd - rd * dn).div_euclid(ld * dn - ln * dd);
				let k2 = (max_den - rd).div_euclid(ld);
				let k = k1.min(k2);
				rn += k * ln;
				rd += k * ld;

				if pl == (ln, ld) && pr == (rn, rd) {
					break;
				}
			}

			let l = Self::new(ln as I, ld) + ipart;
			let r = Self::new(rn as I, rd) + ipart;
			(l, r)
		}
	}

	impl Default for Frac {
		fn default() -> Self {
			Frac { num: 0, den: 1 }
		}
	}

	impl Display for Frac {
		fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
			write!(f, "{}/{}", self.num, self.den)
		}
	}

	impl From<I> for Frac {
		fn from(num: I) -> Self {
			Self { num, den: 1 }
		}
	}

	impl From<Frac> for f64 {
		fn from(value: Frac) -> Self {
			value.num as f64 / value.den as f64
		}
	}

	impl Neg for Frac {
		type Output = Self;
		fn neg(self) -> Self::Output {
			Self { num: -self.num, den: self.den }
		}
	}

	impl Add for Frac {
		type Output = Self;
		fn add(self, rhs: Self) -> Self::Output {
			Self {
				num: self.num * rhs.den as I + self.den as I * rhs.num,
				den: self.den * rhs.den,
			}
			.simplify()
		}
	}

	impl Sub for Frac {
		type Output = Self;
		fn sub(self, rhs: Self) -> Self::Output {
			Self {
				num: self.num * rhs.den as I - self.den as I * rhs.num,
				den: self.den * rhs.den,
			}
			.simplify()
		}
	}

	impl Mul for Frac {
		type Output = Self;
		fn mul(self, rhs: Self) -> Self::Output {
			Self {
				num: self.num * rhs.num,
				den: self.den * rhs.den,
			}
			.simplify()
		}
	}

	impl Div for Frac {
		type Output = Self;
		fn div(self, rhs: Self) -> Self::Output {
			self * rhs.recip()
		}
	}

	impl Add<I> for Frac {
		type Output = Self;
		fn add(self, rhs: I) -> Self::Output {
			let rhs: Frac = rhs.into();
			self + rhs
		}
	}

	impl Sub<I> for Frac {
		type Output = Self;
		fn sub(self, rhs: I) -> Self::Output {
			let rhs: Frac = rhs.into();
			self - rhs
		}
	}

	impl Mul<I> for Frac {
		type Output = Self;
		fn mul(self, rhs: I) -> Self::Output {
			let rhs: Frac = rhs.into();
			self * rhs
		}
	}

	impl Div<I> for Frac {
		type Output = Self;
		fn div(self, rhs: I) -> Self::Output {
			let rhs: Frac = rhs.into();
			self / rhs
		}
	}

	impl Add<Frac> for I {
		type Output = Frac;
		fn add(self, rhs: Frac) -> Self::Output {
			rhs + self
		}
	}

	impl Sub<Frac> for I {
		type Output = Frac;
		fn sub(self, rhs: Frac) -> Self::Output {
			-rhs + self
		}
	}

	impl Mul<Frac> for I {
		type Output = Frac;
		fn mul(self, rhs: Frac) -> Self::Output {
			rhs * self
		}
	}

	impl Div<Frac> for I {
		type Output = Frac;
		fn div(self, rhs: Frac) -> Self::Output {
			let lhs: Frac = self.into();
			lhs / rhs
		}
	}

	impl AddAssign for Frac {
		fn add_assign(&mut self, rhs: Self) {
			*self = *self + rhs;
		}
	}

	impl SubAssign for Frac {
		fn sub_assign(&mut self, rhs: Self) {
			*self = *self - rhs;
		}
	}

	impl MulAssign for Frac {
		fn mul_assign(&mut self, rhs: Self) {
			*self = *self * rhs;
		}
	}

	impl DivAssign for Frac {
		fn div_assign(&mut self, rhs: Self) {
			*self = *self / rhs;
		}
	}

	impl AddAssign<I> for Frac {
		fn add_assign(&mut self, rhs: I) {
			*self = *self + rhs;
		}
	}

	impl SubAssign<I> for Frac {
		fn sub_assign(&mut self, rhs: I) {
			*self = *self - rhs;
		}
	}

	impl MulAssign<I> for Frac {
		fn mul_assign(&mut self, rhs: I) {
			*self = *self * rhs;
		}
	}

	impl DivAssign<I> for Frac {
		fn div_assign(&mut self, rhs: I) {
			*self = *self / rhs;
		}
	}

	impl PartialEq for Frac {
		fn eq(&self, rhs: &Self) -> bool {
			(self.num * rhs.den as I).eq(&(rhs.num * self.den as I))
		}
	}

	impl Eq for Frac {}

	impl PartialOrd for Frac {
		fn partial_cmp(&self, rhs: &Self) -> Option<std::cmp::Ordering> {
			(self.num * rhs.den as I).partial_cmp(&(rhs.num * self.den as I))
		}
	}

	impl Ord for Frac {
		fn cmp(&self, rhs: &Self) -> std::cmp::Ordering {
			(self.num * rhs.den as I).cmp(&(rhs.num * self.den as I))
		}
	}

	impl PartialEq<I> for Frac {
		fn eq(&self, rhs: &I) -> bool {
			let rhs: Frac = (*rhs).into();
			self.eq(&rhs)
		}
	}

	impl PartialOrd<I> for Frac {
		fn partial_cmp(&self, rhs: &I) -> Option<std::cmp::Ordering> {
			let rhs: Frac = (*rhs).into();
			self.partial_cmp(&rhs)
		}
	}

	impl PartialEq<Frac> for I {
		fn eq(&self, rhs: &Frac) -> bool {
			let lhs: Frac = (*self).into();
			lhs.eq(rhs)
		}
	}

	impl PartialOrd<Frac> for I {
		fn partial_cmp(&self, rhs: &Frac) -> Option<std::cmp::Ordering> {
			let lhs: Frac = (*self).into();
			lhs.partial_cmp(rhs)
		}
	}
}
```