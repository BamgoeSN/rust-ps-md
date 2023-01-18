# Bitset

`BitSet` is equivalent to a fixed-size array of booleans. Each boolean value is packed as a bit of `u64`.

For auto-vectorization, each `u64` are packed as `[u64; 4]` so that it can act as a "SIMD lane".

As this snippet is purely for PS and CP, it does not contain many necessary checks, such as checking if two bitset as an argument of a function has the same length.
For any other purpose, I highly recommend the [bitset_core crate](https://docs.rs/bitset-core/latest/bitset_core/) which highly inspired this snippet.

## Usage

An array of `[u64; 4]` implements `BitSetOps` trait, therefore is recognized as a bitset. The number of booleans packed into the bitset can be found with `fn bit_len(&self) -> usize`.

Refer to the [APIs](#apis) for further references.

## Example

```rust
use bitset::*;
# 
# fn main() {
const MAX_VAL: usize = 1000000;

let mut is_prime = [[0u64; 4]; (MAX_VAL + 256) / 256];
println!("{}", is_prime.bit_len()); // 1000192

is_prime.bit_init(true);
is_prime.bit_reset(0);
is_prime.bit_reset(1);

for i in (2..=MAX_VAL).take_while(|&i| i * i <= MAX_VAL) {
    if is_prime.bit_get(i) {
        for j in (i * i..=MAX_VAL).step_by(i) {
            is_prime.bit_reset(j);
        }
    }
}

println!(
    "{}",
    is_prime.bit_count_ones() - (is_prime.bit_len() - (MAX_VAL + 1))
); // 78498
# }
# 
# mod bitset {
#     /* Copyright (c) 2020 Casper <CasualX@users.noreply.github.com>
#      * Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
#      * The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
#      * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#      */
# 
#     //! This module is purely for PS and CP. Thus it skips safety checks such as checking if
#     //! self.len() and rhs.len() are equal, and it may panic if shift overflow (for the whole
#     //! bitset) happens.
# 
#     // DO NOT CHANGE THESE VALUES
#     // The full generalization for bitset is not done.
#     type ElemTy = u64;
#     const ELEM_BIT: usize = ElemTy::BITS as usize;
#     const ELEM_LEN: usize = 4;
#     const BITS_PER_WORD: usize = ELEM_BIT * ELEM_LEN;
# 
#     pub type BitSet = [[ElemTy; ELEM_LEN]];
# 
#     pub trait BitSetOps {
#         fn bit_len(&self) -> usize;
#         fn bit_init(&mut self, val: bool) -> &mut Self;
# 
#         fn bit_get(&self, idx: usize) -> bool;
#         fn bit_set(&mut self, idx: usize) -> &mut Self;
#         fn bit_reset(&mut self, idx: usize) -> &mut Self;
#         fn bit_flip(&mut self, idx: usize) -> &mut Self;
#         fn bit_manip(&mut self, idx: usize, val: bool) -> &mut Self;
# 
#         fn bit_all(&self) -> bool;
#         fn bit_any(&self) -> bool;
#         #[inline]
#         fn bit_none(&self) -> bool {
#             !self.bit_any()
#         }
# 
#         fn bit_eq(&self, rhs: &Self) -> bool;
#         fn bit_disjoint(&self, rhs: &Self) -> bool;
#         fn bit_subset(&self, rhs: &Self) -> bool;
#         #[inline]
#         fn bit_superset(&self, rhs: &Self) -> bool {
#             rhs.bit_subset(self)
#         }
# 
#         fn bit_or(&mut self, rhs: &Self) -> &mut Self;
#         fn bit_and(&mut self, rhs: &Self) -> &mut Self;
#         fn bit_nand(&mut self, rhs: &Self) -> &mut Self;
#         fn bit_xor(&mut self, rhs: &Self) -> &mut Self;
#         fn bit_not(&mut self) -> &mut Self;
#         fn bit_mask(&mut self, rhs: &Self, mask: &Self) -> &mut Self;
# 
#         fn bit_shr(&mut self, by: usize) -> &mut Self;
#         fn bit_shl(&mut self, by: usize) -> &mut Self;
# 
#         fn bit_count_ones(&self) -> usize;
#         #[inline]
#         fn bit_count_zeros(&self) -> usize {
#             self.bit_len() - self.bit_count_ones()
#         }
# 
#         #[inline]
#         fn bit_fmt(&self) -> &BitFmt<Self> {
#             unsafe { &*(self as *const _ as *const _) }
#         }
#     }
# 
#     impl BitSetOps for BitSet {
#         #[inline]
#         fn bit_len(&self) -> usize {
#             self.len() * BITS_PER_WORD
#         }
# 
#         #[inline]
#         fn bit_init(&mut self, val: bool) -> &mut Self {
#             let val = [ElemTy::wrapping_add(!(val as ElemTy), 1); ELEM_LEN];
#             for i in 0..self.len() {
#                 self[i] = val;
#             }
#             self
#         }
# 
#         #[inline]
#         fn bit_get(&self, idx: usize) -> bool {
#             let block = idx / BITS_PER_WORD;
#             let lane = (idx % BITS_PER_WORD) / ELEM_BIT;
#             let bit = idx % ELEM_BIT;
#             self[block][lane] & (1 << bit) != 0
#         }
# 
#         #[inline]
#         fn bit_set(&mut self, idx: usize) -> &mut Self {
#             let block = idx / BITS_PER_WORD;
#             let lane = (idx % BITS_PER_WORD) / ELEM_BIT;
#             let bit = idx % ELEM_BIT;
#             self[block][lane] |= 1 << bit;
#             self
#         }
# 
#         #[inline]
#         fn bit_reset(&mut self, idx: usize) -> &mut Self {
#             let block = idx / BITS_PER_WORD;
#             let lane = (idx % BITS_PER_WORD) / ELEM_BIT;
#             let bit = idx % ELEM_BIT;
#             self[block][lane] &= !(1 << bit);
#             self
#         }
# 
#         #[inline]
#         fn bit_flip(&mut self, idx: usize) -> &mut Self {
#             let block = idx / BITS_PER_WORD;
#             let lane = (idx % BITS_PER_WORD) / ELEM_BIT;
#             let bit = idx % ELEM_BIT;
#             self[block][lane] ^= 1 << bit;
#             self
#         }
# 
#         #[inline]
#         fn bit_manip(&mut self, idx: usize, val: bool) -> &mut Self {
#             let block = idx / BITS_PER_WORD;
#             let lane = (idx % BITS_PER_WORD) / ELEM_BIT;
#             let bit = idx % ELEM_BIT;
#             let mask = 1 << bit;
#             self[block][lane] =
#                 (self[block][lane] & !mask) | (ElemTy::wrapping_add(!(val as ElemTy), 1) & mask);
#             self
#         }
# 
#         #[inline]
#         fn bit_all(&self) -> bool {
#             self.iter()
#                 .all(|block| block[0] == !0 && block[1] == !0 && block[2] == !0 && block[3] == !0)
#         }
# 
#         #[inline]
#         fn bit_any(&self) -> bool {
#             self.iter()
#                 .all(|block| block[0] == 0 && block[1] == 0 && block[2] == 0 && block[3] == 0)
#         }
# 
#         #[inline]
#         fn bit_eq(&self, rhs: &Self) -> bool {
#             self.iter().zip(rhs.iter()).all(|(&lblk, &rblk)| {
#                 lblk[0] == rblk[0] && lblk[1] == rblk[1] && lblk[2] == rblk[2] && lblk[3] == rblk[3]
#             })
#         }
# 
#         #[inline]
#         fn bit_disjoint(&self, rhs: &Self) -> bool {
#             self.iter().zip(rhs.iter()).all(|(&lblk, &rblk)| {
#                 lblk[0] & rblk[0] == 0
#                     && lblk[1] & rblk[1] == 0
#                     && lblk[2] & rblk[2] == 0
#                     && lblk[3] & rblk[3] == 0
#             })
#         }
# 
#         #[inline]
#         fn bit_subset(&self, rhs: &Self) -> bool {
#             self.iter().zip(rhs.iter()).all(|(&lblk, &rblk)| {
#                 rblk[0] == rblk[0] | lblk[0]
#                     && rblk[1] == rblk[1] | lblk[1]
#                     && rblk[2] == rblk[2] | lblk[2]
#                     && rblk[3] == rblk[3] | lblk[3]
#             })
#         }
# 
#         #[inline]
#         fn bit_or(&mut self, rhs: &Self) -> &mut Self {
#             for i in 0..self.len() {
#                 self[i][0] |= rhs[i][0];
#                 self[i][1] |= rhs[i][1];
#                 self[i][2] |= rhs[i][2];
#                 self[i][3] |= rhs[i][3];
#             }
#             self
#         }
# 
#         #[inline]
#         fn bit_and(&mut self, rhs: &Self) -> &mut Self {
#             for i in 0..self.len() {
#                 self[i][0] &= rhs[i][0];
#                 self[i][1] &= rhs[i][1];
#                 self[i][2] &= rhs[i][2];
#                 self[i][3] &= rhs[i][3];
#             }
#             self
#         }
# 
#         #[inline]
#         fn bit_nand(&mut self, rhs: &Self) -> &mut Self {
#             for i in 0..self.len() {
#                 self[i][0] &= !rhs[i][0];
#                 self[i][1] &= !rhs[i][1];
#                 self[i][2] &= !rhs[i][2];
#                 self[i][3] &= !rhs[i][3];
#             }
#             self
#         }
# 
#         #[inline]
#         fn bit_xor(&mut self, rhs: &Self) -> &mut Self {
#             for i in 0..self.len() {
#                 self[i][0] ^= rhs[i][0];
#                 self[i][1] ^= rhs[i][1];
#                 self[i][2] ^= rhs[i][2];
#                 self[i][3] ^= rhs[i][3];
#             }
#             self
#         }
# 
#         #[inline]
#         fn bit_not(&mut self) -> &mut Self {
#             for i in 0..self.len() {
#                 self[i][0] = !self[i][0];
#                 self[i][1] = !self[i][1];
#                 self[i][2] = !self[i][2];
#                 self[i][3] = !self[i][3];
#             }
#             self
#         }
# 
#         #[inline]
#         fn bit_mask(&mut self, rhs: &Self, mask: &Self) -> &mut Self {
#             for i in 0..self.len() {
#                 self[i][0] = self[i][0] & !mask[i][0] | rhs[i][0] & mask[i][0];
#                 self[i][1] = self[i][1] & !mask[i][1] | rhs[i][1] & mask[i][1];
#                 self[i][2] = self[i][2] & !mask[i][2] | rhs[i][2] & mask[i][2];
#                 self[i][3] = self[i][3] & !mask[i][3] | rhs[i][3] & mask[i][3];
#             }
#             self
#         }
# 
#         #[inline]
#         fn bit_shl(&mut self, by: usize) -> &mut Self {
#             let elem_move = by / ELEM_BIT;
#             let bit_move = by % ELEM_BIT;
# 
#             let (_, slice, _): (_, &mut [ElemTy], _) = unsafe { self.align_to_mut() };
#             slice.copy_within(..slice.len() - elem_move, elem_move);
#             slice[..elem_move].fill(0);
# 
#             if bit_move != 0 {
#                 let mut carry: ElemTy = 0;
#                 let mut tmp: [ElemTy; ELEM_LEN] = [0; ELEM_LEN];
#                 for i in 0..self.len() {
#                     tmp[0] = self[i][0] >> (ELEM_BIT - bit_move);
#                     tmp[1] = self[i][1] >> (ELEM_BIT - bit_move);
#                     tmp[2] = self[i][2] >> (ELEM_BIT - bit_move);
#                     tmp[3] = self[i][3] >> (ELEM_BIT - bit_move);
#                     self[i][0] <<= bit_move;
#                     self[i][1] <<= bit_move;
#                     self[i][2] <<= bit_move;
#                     self[i][3] <<= bit_move;
#                     let tmpc = tmp[ELEM_LEN - 1];
#                     tmp.copy_within(..ELEM_LEN - 1, 1);
#                     tmp[0] = carry;
#                     self[i][0] |= tmp[0];
#                     self[i][1] |= tmp[1];
#                     self[i][2] |= tmp[2];
#                     self[i][3] |= tmp[3];
#                     carry = tmpc;
#                 }
#             }
# 
#             self
#         }
# 
#         #[inline]
#         fn bit_shr(&mut self, by: usize) -> &mut Self {
#             let elem_move = by / ELEM_BIT;
#             let bit_move = by % ELEM_BIT;
# 
#             let (_, slice, _): (_, &mut [ElemTy], _) = unsafe { self.align_to_mut() };
#             slice.copy_within(elem_move.., 0);
#             let sl = slice.len();
#             slice[sl - elem_move..].fill(0);
# 
#             if bit_move != 0 {
#                 let mut carry: ElemTy = 0;
#                 let mut tmp: [ElemTy; ELEM_LEN] = [0; ELEM_LEN];
#                 for i in 0..self.len() {
#                     tmp[0] = self[i][0] << (ELEM_BIT - bit_move);
#                     tmp[1] = self[i][1] << (ELEM_BIT - bit_move);
#                     tmp[2] = self[i][2] << (ELEM_BIT - bit_move);
#                     tmp[3] = self[i][3] << (ELEM_BIT - bit_move);
#                     self[i][0] >>= bit_move;
#                     self[i][1] >>= bit_move;
#                     self[i][2] >>= bit_move;
#                     self[i][3] >>= bit_move;
#                     let tmpc = tmp[0];
#                     tmp.copy_within(1.., 0);
#                     tmp[ELEM_LEN - 1] = carry;
#                     carry = tmpc;
#                     self[i][0] |= tmp[0];
#                     self[i][1] |= tmp[1];
#                     self[i][2] |= tmp[2];
#                     self[i][3] |= tmp[3];
#                 }
#             }
# 
#             self
#         }
# 
#         #[inline]
#         fn bit_count_ones(&self) -> usize {
#             self.iter()
#                 .map(|chunk| {
#                     chunk[0].count_ones() as usize
#                         + chunk[1].count_ones() as usize
#                         + chunk[2].count_ones() as usize
#                         + chunk[3].count_ones() as usize
#                 })
#                 .sum()
#         }
#     }
# 
#     mod fmt {
#         use super::BitSetOps as BitSet;
#         use std::fmt;
# 
#         #[repr(transparent)]
#         pub struct BitFmt<T: ?Sized>(T);
# 
#         fn bitstring<T: ?Sized + BitSet>(this: &T, f: &mut fmt::Formatter) -> fmt::Result {
#             const ALPHABET: [u8; 2] = [b'0', b'1'];
#             let mut buf = [0u8; 9];
#             let mut first = true;
#             buf[0] = b'_';
#             let mut i = 0;
#             while i < this.bit_len() {
#                 buf[1] = ALPHABET[this.bit_get(i + 0) as usize];
#                 buf[2] = ALPHABET[this.bit_get(i + 1) as usize];
#                 buf[3] = ALPHABET[this.bit_get(i + 2) as usize];
#                 buf[4] = ALPHABET[this.bit_get(i + 3) as usize];
#                 buf[5] = ALPHABET[this.bit_get(i + 4) as usize];
#                 buf[6] = ALPHABET[this.bit_get(i + 5) as usize];
#                 buf[7] = ALPHABET[this.bit_get(i + 6) as usize];
#                 buf[8] = ALPHABET[this.bit_get(i + 7) as usize];
#                 let s = unsafe { &*((&buf[first as usize..]) as *const _ as *const str) };
#                 f.write_str(s)?;
#                 i += 8;
#                 first = false;
#             }
#             Ok(())
#         }
# 
#         impl<T: ?Sized + BitSet> fmt::Display for BitFmt<T> {
#             fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
#                 bitstring(&self.0, f)
#             }
#         }
#     }
#     pub use self::fmt::BitFmt;
# }
```

## Code

```rust,noplayground
mod bitset {
    /* Copyright (c) 2020 Casper <CasualX@users.noreply.github.com>
     * Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
     * The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
     * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
     */

    //! This module is purely for PS and CP. Thus it skips safety checks such as checking if
    //! self.len() and rhs.len() are equal, and it may panic if shift overflow (for the whole
    //! bitset) happens.

    // DO NOT CHANGE THESE VALUES
    // The full generalization for bitset is not done.
    type ElemTy = u64;
    const ELEM_BIT: usize = ElemTy::BITS as usize;
    const ELEM_LEN: usize = 4;
    const BITS_PER_WORD: usize = ELEM_BIT * ELEM_LEN;

    pub type BitSet = [[ElemTy; ELEM_LEN]];

    pub trait BitSetOps {
        fn bit_len(&self) -> usize;
        fn bit_init(&mut self, val: bool) -> &mut Self;

        fn bit_get(&self, idx: usize) -> bool;
        fn bit_set(&mut self, idx: usize) -> &mut Self;
        fn bit_reset(&mut self, idx: usize) -> &mut Self;
        fn bit_flip(&mut self, idx: usize) -> &mut Self;
        fn bit_manip(&mut self, idx: usize, val: bool) -> &mut Self;

        fn bit_all(&self) -> bool;
        fn bit_any(&self) -> bool;
        #[inline]
        fn bit_none(&self) -> bool {
            !self.bit_any()
        }

        fn bit_eq(&self, rhs: &Self) -> bool;
        fn bit_disjoint(&self, rhs: &Self) -> bool;
        fn bit_subset(&self, rhs: &Self) -> bool;
        #[inline]
        fn bit_superset(&self, rhs: &Self) -> bool {
            rhs.bit_subset(self)
        }

        fn bit_or(&mut self, rhs: &Self) -> &mut Self;
        fn bit_and(&mut self, rhs: &Self) -> &mut Self;
        fn bit_nand(&mut self, rhs: &Self) -> &mut Self;
        fn bit_xor(&mut self, rhs: &Self) -> &mut Self;
        fn bit_not(&mut self) -> &mut Self;
        fn bit_mask(&mut self, rhs: &Self, mask: &Self) -> &mut Self;

        fn bit_shr(&mut self, by: usize) -> &mut Self;
        fn bit_shl(&mut self, by: usize) -> &mut Self;

        fn bit_count_ones(&self) -> usize;
        #[inline]
        fn bit_count_zeros(&self) -> usize {
            self.bit_len() - self.bit_count_ones()
        }

        #[inline]
        fn bit_fmt(&self) -> &BitFmt<Self> {
            unsafe { &*(self as *const _ as *const _) }
        }
    }

    impl BitSetOps for BitSet {
        #[inline]
        fn bit_len(&self) -> usize {
            self.len() * BITS_PER_WORD
        }

        #[inline]
        fn bit_init(&mut self, val: bool) -> &mut Self {
            let val = [ElemTy::wrapping_add(!(val as ElemTy), 1); ELEM_LEN];
            for i in 0..self.len() {
                self[i] = val;
            }
            self
        }

        #[inline]
        fn bit_get(&self, idx: usize) -> bool {
            let block = idx / BITS_PER_WORD;
            let lane = (idx % BITS_PER_WORD) / ELEM_BIT;
            let bit = idx % ELEM_BIT;
            self[block][lane] & (1 << bit) != 0
        }

        #[inline]
        fn bit_set(&mut self, idx: usize) -> &mut Self {
            let block = idx / BITS_PER_WORD;
            let lane = (idx % BITS_PER_WORD) / ELEM_BIT;
            let bit = idx % ELEM_BIT;
            self[block][lane] |= 1 << bit;
            self
        }

        #[inline]
        fn bit_reset(&mut self, idx: usize) -> &mut Self {
            let block = idx / BITS_PER_WORD;
            let lane = (idx % BITS_PER_WORD) / ELEM_BIT;
            let bit = idx % ELEM_BIT;
            self[block][lane] &= !(1 << bit);
            self
        }

        #[inline]
        fn bit_flip(&mut self, idx: usize) -> &mut Self {
            let block = idx / BITS_PER_WORD;
            let lane = (idx % BITS_PER_WORD) / ELEM_BIT;
            let bit = idx % ELEM_BIT;
            self[block][lane] ^= 1 << bit;
            self
        }

        #[inline]
        fn bit_manip(&mut self, idx: usize, val: bool) -> &mut Self {
            let block = idx / BITS_PER_WORD;
            let lane = (idx % BITS_PER_WORD) / ELEM_BIT;
            let bit = idx % ELEM_BIT;
            let mask = 1 << bit;
            self[block][lane] =
                (self[block][lane] & !mask) | (ElemTy::wrapping_add(!(val as ElemTy), 1) & mask);
            self
        }

        #[inline]
        fn bit_all(&self) -> bool {
            self.iter()
                .all(|block| block[0] == !0 && block[1] == !0 && block[2] == !0 && block[3] == !0)
        }

        #[inline]
        fn bit_any(&self) -> bool {
            self.iter()
                .all(|block| block[0] == 0 && block[1] == 0 && block[2] == 0 && block[3] == 0)
        }

        #[inline]
        fn bit_eq(&self, rhs: &Self) -> bool {
            self.iter().zip(rhs.iter()).all(|(&lblk, &rblk)| {
                lblk[0] == rblk[0] && lblk[1] == rblk[1] && lblk[2] == rblk[2] && lblk[3] == rblk[3]
            })
        }

        #[inline]
        fn bit_disjoint(&self, rhs: &Self) -> bool {
            self.iter().zip(rhs.iter()).all(|(&lblk, &rblk)| {
                lblk[0] & rblk[0] == 0
                    && lblk[1] & rblk[1] == 0
                    && lblk[2] & rblk[2] == 0
                    && lblk[3] & rblk[3] == 0
            })
        }

        /// Returns if self is a subset of rhs
        #[inline]
        fn bit_subset(&self, rhs: &Self) -> bool {
            self.iter().zip(rhs.iter()).all(|(&lblk, &rblk)| {
                rblk[0] == rblk[0] | lblk[0]
                    && rblk[1] == rblk[1] | lblk[1]
                    && rblk[2] == rblk[2] | lblk[2]
                    && rblk[3] == rblk[3] | lblk[3]
            })
        }

        #[inline]
        fn bit_or(&mut self, rhs: &Self) -> &mut Self {
            for i in 0..self.len() {
                self[i][0] |= rhs[i][0];
                self[i][1] |= rhs[i][1];
                self[i][2] |= rhs[i][2];
                self[i][3] |= rhs[i][3];
            }
            self
        }

        #[inline]
        fn bit_and(&mut self, rhs: &Self) -> &mut Self {
            for i in 0..self.len() {
                self[i][0] &= rhs[i][0];
                self[i][1] &= rhs[i][1];
                self[i][2] &= rhs[i][2];
                self[i][3] &= rhs[i][3];
            }
            self
        }

        #[inline]
        fn bit_nand(&mut self, rhs: &Self) -> &mut Self {
            for i in 0..self.len() {
                self[i][0] &= !rhs[i][0];
                self[i][1] &= !rhs[i][1];
                self[i][2] &= !rhs[i][2];
                self[i][3] &= !rhs[i][3];
            }
            self
        }

        #[inline]
        fn bit_xor(&mut self, rhs: &Self) -> &mut Self {
            for i in 0..self.len() {
                self[i][0] ^= rhs[i][0];
                self[i][1] ^= rhs[i][1];
                self[i][2] ^= rhs[i][2];
                self[i][3] ^= rhs[i][3];
            }
            self
        }

        #[inline]
        fn bit_not(&mut self) -> &mut Self {
            for i in 0..self.len() {
                self[i][0] = !self[i][0];
                self[i][1] = !self[i][1];
                self[i][2] = !self[i][2];
                self[i][3] = !self[i][3];
            }
            self
        }

        #[inline]
        fn bit_mask(&mut self, rhs: &Self, mask: &Self) -> &mut Self {
            for i in 0..self.len() {
                self[i][0] = self[i][0] & !mask[i][0] | rhs[i][0] & mask[i][0];
                self[i][1] = self[i][1] & !mask[i][1] | rhs[i][1] & mask[i][1];
                self[i][2] = self[i][2] & !mask[i][2] | rhs[i][2] & mask[i][2];
                self[i][3] = self[i][3] & !mask[i][3] | rhs[i][3] & mask[i][3];
            }
            self
        }

        #[inline]
        fn bit_shl(&mut self, by: usize) -> &mut Self {
            let elem_move = by / ELEM_BIT;
            let bit_move = by % ELEM_BIT;

            let (_, slice, _): (_, &mut [ElemTy], _) = unsafe { self.align_to_mut() };
            slice.copy_within(..slice.len() - elem_move, elem_move);
            slice[..elem_move].fill(0);

            if bit_move != 0 {
                let mut carry: ElemTy = 0;
                let mut tmp: [ElemTy; ELEM_LEN] = [0; ELEM_LEN];
                for i in 0..self.len() {
                    tmp[0] = self[i][0] >> (ELEM_BIT - bit_move);
                    tmp[1] = self[i][1] >> (ELEM_BIT - bit_move);
                    tmp[2] = self[i][2] >> (ELEM_BIT - bit_move);
                    tmp[3] = self[i][3] >> (ELEM_BIT - bit_move);
                    self[i][0] <<= bit_move;
                    self[i][1] <<= bit_move;
                    self[i][2] <<= bit_move;
                    self[i][3] <<= bit_move;
                    let tmpc = tmp[ELEM_LEN - 1];
                    tmp.copy_within(..ELEM_LEN - 1, 1);
                    tmp[0] = carry;
                    self[i][0] |= tmp[0];
                    self[i][1] |= tmp[1];
                    self[i][2] |= tmp[2];
                    self[i][3] |= tmp[3];
                    carry = tmpc;
                }
            }

            self
        }

        #[inline]
        fn bit_shr(&mut self, by: usize) -> &mut Self {
            let elem_move = by / ELEM_BIT;
            let bit_move = by % ELEM_BIT;

            let (_, slice, _): (_, &mut [ElemTy], _) = unsafe { self.align_to_mut() };
            slice.copy_within(elem_move.., 0);
            let sl = slice.len();
            slice[sl - elem_move..].fill(0);

            if bit_move != 0 {
                let mut carry: ElemTy = 0;
                let mut tmp: [ElemTy; ELEM_LEN] = [0; ELEM_LEN];
                for i in 0..self.len() {
                    tmp[0] = self[i][0] << (ELEM_BIT - bit_move);
                    tmp[1] = self[i][1] << (ELEM_BIT - bit_move);
                    tmp[2] = self[i][2] << (ELEM_BIT - bit_move);
                    tmp[3] = self[i][3] << (ELEM_BIT - bit_move);
                    self[i][0] >>= bit_move;
                    self[i][1] >>= bit_move;
                    self[i][2] >>= bit_move;
                    self[i][3] >>= bit_move;
                    let tmpc = tmp[0];
                    tmp.copy_within(1.., 0);
                    tmp[ELEM_LEN - 1] = carry;
                    carry = tmpc;
                    self[i][0] |= tmp[0];
                    self[i][1] |= tmp[1];
                    self[i][2] |= tmp[2];
                    self[i][3] |= tmp[3];
                }
            }

            self
        }

        #[inline]
        fn bit_count_ones(&self) -> usize {
            self.iter()
                .map(|chunk| {
                    chunk[0].count_ones() as usize
                        + chunk[1].count_ones() as usize
                        + chunk[2].count_ones() as usize
                        + chunk[3].count_ones() as usize
                })
                .sum()
        }
    }

    mod fmt {
        use super::BitSetOps as BitSet;
        use std::fmt;

        #[repr(transparent)]
        pub struct BitFmt<T: ?Sized>(T);

        fn bitstring<T: ?Sized + BitSet>(this: &T, f: &mut fmt::Formatter) -> fmt::Result {
            const ALPHABET: [u8; 2] = [b'0', b'1'];
            let mut buf = [0u8; 9];
            let mut first = true;
            buf[0] = b'_';
            let mut i = 0;
            while i < this.bit_len() {
                buf[1] = ALPHABET[this.bit_get(i + 0) as usize];
                buf[2] = ALPHABET[this.bit_get(i + 1) as usize];
                buf[3] = ALPHABET[this.bit_get(i + 2) as usize];
                buf[4] = ALPHABET[this.bit_get(i + 3) as usize];
                buf[5] = ALPHABET[this.bit_get(i + 4) as usize];
                buf[6] = ALPHABET[this.bit_get(i + 5) as usize];
                buf[7] = ALPHABET[this.bit_get(i + 6) as usize];
                buf[8] = ALPHABET[this.bit_get(i + 7) as usize];
                let s = unsafe { &*((&buf[first as usize..]) as *const _ as *const str) };
                f.write_str(s)?;
                i += 8;
                first = false;
            }
            Ok(())
        }

        impl<T: ?Sized + BitSet> fmt::Display for BitFmt<T> {
            fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
                bitstring(&self.0, f)
            }
        }
    }
    pub use self::fmt::BitFmt;
}
```

## SIMD, Auto-vectorization and `rustc` Optimization Level

To fully enable the power of aggressive SIMD optimization, the `opt-level` for compilation should be 3.
When the level is 2, despite many of vectorizations still happen, the occurence noticably decreases.

As most of the OJs compile Rust codes with `opt-level` of 2, to fully enable the power of SIMD, you need to hardcode the machine code into Rust.
As this is virtually impossible to do manually in actual PS/CP, using third-party tools like [basm-rs](https://github.com/kiwiyou/basm-rs) is highly recommended.

## APIs

The behavior of APIs having multiple bitsets as arguments, when their length are not equal to each other, is unspecified.

- `pub type BitSet`

  `[u64; 4]` is defined as a `type BitSet`.
  As this is a dynamically sized type, when declaring a bitset you cannot use `BitSet` in its type declaration.
  Instead, you need to do like the below example.
  ```rust,noplayground
  let mut bitset
  ```

- `fn bit_len(&self) -> usize`

  Returns the number of boolean values included in the set.

- `fn bit_init(&mut self, val: bool) -> &mut Self`

  Initializes every boolean value of `self` as `val`, and returns `&mut self` back.

- `fn bit_get(&self, idx: usize) -> bool`

  Returns the `idx`th boolean value of `self`.

- `fn bit_set(&mut self, idx: usize) -> &mut Self`

  Sets the `idx`th boolean value to `true`, and returns `&mut self` back.

- `fn bit_reset(&mut self, idx: usize) -> &mut Self`

  Sets the `idx`th boolean value to `false`, and returns `&mut self` back.

- `fn bit_flip(&mut self, idx: usize) -> &mut Self`

  Flips the `idx`th boolean value, and returns `&mut self` back.

- `fn bit_manip(&mut self, idx: usize, val: bool) -> &mut Self`

  Sets the `idx`th boolean value to `val`, and returns `&mut self` back.

- `fn bit_all(&self) -> bool`

  Returns `true` if every boolean value of `self` is `true`. Otherwise, returns `false`.

- `fn bit_none(&self) -> bool`

  Returns `true` if every boolean value of `self` is `false`. Otherwise, returns `false`.

- `fn bit_disjoint(&self, rhs: &Self) -> bool`

  Returns `true` if every bit of `self` turned on is not in `rhs`, and vice versa. Otherwise, returns `false`.

- `fn bit_subset(&self, rhs: &Self) -> bool`
  
  Returns `true` if `self` is a subset of `rhs`. Otherwise, returns `false`.

- `fn bit_superset(&self, rhs: &Self) -> bool`

  Returns `true` if `self` is a superset of `rhs`. Otherwise, returns `false`.

- `fn bit_or(&mut self, rhs: &Self) -> &mut Self`

  Sets `self` as `self | rhs`, and returns `&mut self` back.

- `fn bit_and(&mut self, rhs: &Self) -> &mut Self`

  Sets `self` as `self & rhs`, and returns `&mut self` back.

- `fn bit_nand(&mut self, rhs: &Self) -> &mut Self`

  Sets `self` as `self & !rhs`, and returns `&mut self` back.

- `fn bit_xor(&mut self, rhs: &Self) -> &mut Self`

  Sets `self` as `self ^ rhs`, and returns `&mut self` back.

- `fn bit_not(&mut self) -> &mut Self`

  Reverses every bits of `self`, and returns `&mut self` back.

- `fn bit_mask(&mut self, rhs: &Self, mask: &Self) -> &mut Self`

  Sets `self` as `(self & !mask) | (rhs & mask)`, and returns `&mut self` back.

- `fn bit_shr(&mut self, by: usize) -> &mut Self`

  Shifts `self` right by `by`. The direction of shifting is to the lower index.
  The empty bits are filled with `0`, and the overflowed bits disappear.

- `fn bit_shl(&mut self, by: usize) -> &mut Self`

  Shifts `self` left by `by`. The direction of shifting is to the lower index.
  The empty bits are filled with `0`, and the overflowed bits disappear.

- `fn bit_count_ones(&self) -> usize`

  Returns the number of boolean values that is `true`.

- `fn bit_count_zeros(&self) -> usize`

  Returns the number of boolean values that is `false`.

- `fn bit_fmt(&self) -> &BitFmt<Self>`

  Used for printing out the bitset.
  ```rust,noplayground
  println!("{}", bitset.bit_fmt());
  ```
