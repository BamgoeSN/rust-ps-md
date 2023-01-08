# Rope

Rope acts as if it is a list, but inserting a value at an arbitrary position takes time complexity of amortized \\( O(\log{N}) \\). However, accessing values also takes amortized \\( O(\log{N}) \\) time. Building a rope from an iterator takes \\( O(N) \\).

## Example
```rust
# fn main() {
use rope::Rope;

let mut arr: Rope<i32> = (0..10).collect();
println!("{:?}", arr);

let out = arr.take_range(1..5).unwrap();
arr.merge_right(out);
println!("{:?}", arr);

for i in 11..100000 {
    let n = arr.len() / 2;
    arr.insert(n, i);
}
println!("{}", arr[50000]);

for _ in 0..arr.len() - 10 {
    let n = arr.len() / 2;
    arr.remove(n + 1);
}
println!("{:?}", arr);
# }
# mod rope {
#     use std::{
#         cmp::Ordering,
#         fmt::{Debug, Display},
#         ops::{Bound::*, Index, IndexMut, RangeBounds},
#         ptr::{self, NonNull},
#         iter::FromIterator,
#     };
#     pub struct Node<T> {
#         data: T,
#         subt: usize,
#         l: Link<T>,
#         r: Link<T>,
#         p: Link<T>,
#     }
#     type Link<T> = Option<NonNull<Node<T>>>;
#     impl<T> Node<T> {
#         fn new(data: T) -> Self {
#             Node {
#                 data,
#                 subt: 1,
#                 l: None,
#                 r: None,
#                 p: None,
#             }
#         }
#         fn left_size(&self) -> usize {
#             unsafe { self.l.map_or(0, |l| (*l.as_ptr()).subt) }
#         }
#         fn right_size(&self) -> usize {
#             unsafe { self.r.map_or(0, |r| (*r.as_ptr()).subt) }
#         }
#         fn upd_subtree(&mut self) {
#             self.subt = 1 + self.left_size() + self.right_size();
#         }
#         // Option<(is_left, parent)>
#         unsafe fn is_left_child(x: NonNull<Self>) -> Option<(bool, NonNull<Self>)> {
#             if let Some(p) = (*x.as_ptr()).p {
#                 if (*p.as_ptr())
#                     .l
#                     .map_or(false, |pl| ptr::eq(x.as_ptr(), pl.as_ptr()))
#                 {
#                     Some((true, p))
#                 } else {
#                     Some((false, p))
#                 }
#             } else {
#                 None
#             }
#         }
#     }
#     pub struct Rope<T> {
#         root: Link<T>,
#         size: usize,
#     }
#     impl<T> Default for Rope<T> {
#         fn default() -> Self {
#             Self {
#                 root: None,
#                 size: 0,
#             }
#         }
#     }
#     impl<T> Rope<T> {
#         pub fn new() -> Self {
#             Self::default()
#         }
#         pub fn len(&self) -> usize {
#             self.size
#         }
#         pub fn insert(&mut self, idx: usize, data: T) {
#             debug_assert!(idx <= self.size);
#             unsafe {
#                 let new_node = NonNull::new_unchecked(Box::into_raw(Box::new(Node::new(data))));
#                 if let Some(r) = self.root {
#                     let idx = self.kth_ptr(idx);
#                     if let Some(idx) = idx {
#                         // idx_node is the node which new_node should replace
#                         // "Replace" means the new_node should be placed right before the idx_node
#                         if let Some(l) = (*idx.as_ptr()).l {
#                             // Attach at the right of rightmost node from l
#                             let mut p = l;
#                             while let Some(r) = (*p.as_ptr()).r {
#                                 p = r;
#                             }
#                             // Attach new_node to the right of p
#                             (*new_node.as_ptr()).p = Some(p);
#                             (*p.as_ptr()).r = Some(new_node);
#                         } else {
#                             // Attach it right away
#                             let p = idx;
#                             (*new_node.as_ptr()).p = Some(p);
#                             (*p.as_ptr()).l = Some(new_node);
#                         }
#                     } else {
#                         // idx == self.size
#                         // new_node goes to the rightmost of the tree
#                         let mut p = r;
#                         while let Some(r) = (*p.as_ptr()).r {
#                             p = r;
#                         }
#                         // Attach new_node to the right of p
#                         (*new_node.as_ptr()).p = Some(p);
#                         (*p.as_ptr()).r = Some(new_node);
#                     }
#                     let mut c = new_node;
#                     while let Some(p) = (*c.as_ptr()).p {
#                         c = p;
#                         (*c.as_ptr()).upd_subtree();
#                     }
#                 } else {
#                     self.root = Some(new_node);
#                 }
#                 self.splay(new_node);
#                 self.size += 1;
#             }
#         }
#         pub fn remove(&mut self, idx: usize) -> Option<T> {
#             if idx >= self.size {
#                 return None;
#             }
#             let data: T = unsafe {
#                 if let Some(mut rt) = self.kth_ptr(idx) {
#                     rt = self.remove_helper(rt);
#                     if let Some(rp) = (*rt.as_ptr()).p {
#                         self.splay(rp);
#                     }
#                     let retr = Box::from_raw(rt.as_ptr());
#                     retr.data
#                 } else {
#                     unreachable!()
#                 }
#             };
#             self.size -= 1;
#             Some(data)
#         }
#         pub fn push_front(&mut self, data: T) {
#             self.insert(0, data);
#         }
#         pub fn push_back(&mut self, data: T) {
#             self.insert(self.size, data);
#         }
#         pub fn pop_front(&mut self) -> Option<T> {
#             self.remove(0)
#         }
#         pub fn pop_back(&mut self) -> Option<T> {
#             self.remove(self.size - 1)
#         }
#         /// Splits out the rope, leaving self[..at] and returning self[at..].
#         /// If the index is invalid, it returns None.
#         pub fn take_right(&mut self, right_start: usize) -> Option<Self> {
#             let rhs = unsafe {
#                 if right_start == 0 {
#                     let rhs = Self {
#                         root: self.root,
#                         size: self.size,
#                     };
#                     self.root = None;
#                     self.size = 0;
#                     rhs
#                 } else {
#                     let root = self.kth_ptr(right_start - 1)?;
#                     self.splay(root);
#                     if let Some(r) = (*root.as_ptr()).r {
#                         (*root.as_ptr()).r = None;
#                         (*r.as_ptr()).p = None;
#                         (*root.as_ptr()).upd_subtree();
#                         self.size = (*root.as_ptr()).subt;
#                         Self {
#                             root: Some(r),
#                             size: (*r.as_ptr()).subt,
#                         }
#                     } else {
#                         Self {
#                             root: None,
#                             size: 0,
#                         }
#                     }
#                 }
#             };
#             Some(rhs)
#         }
#         /// Splits out the rope and returns self[..at] and self[at..].
#         /// If the index is invalid, it returns None.
#         pub fn split_at(mut self, at: usize) -> Option<(Self, Self)> {
#             let rhs = self.take_right(at)?;
#             Some((self, rhs))
#         }
#         /// Takes out the range from the rope.
#         /// Returns None if the index is invalid.
#         pub fn take_range(&mut self, range: impl RangeBounds<usize>) -> Option<Self> {
#             let l = match range.start_bound() {
#                 Included(&l) => l,
#                 Excluded(&l) => l + 1,
#                 Unbounded => 0,
#             };
#             let r = match range.end_bound() {
#                 Included(&r) => r + 1,
#                 Excluded(&r) => r,
#                 Unbounded => self.size,
#             };
#             if l > r || l > self.size || r > self.size {
#                 return None;
#             }
#             // Now the operations below never ends early
#             let c = self.take_right(r)?;
#             let b = self.take_right(l)?;
#             self.merge_right(c);
#             Some(b)
#         }
#         pub fn merge_right(&mut self, mut rhs: Self) {
#             if self.len() == 0 {
#                 self.root = rhs.root;
#                 self.size = rhs.size;
#             } else {
#                 unsafe {
#                     let rmost = self.kth_ptr(self.size - 1).unwrap();
#                     self.splay(rmost);
#                     (*rmost.as_ptr()).r = rhs.root;
#                     if let Some(rhs_root) = rhs.root {
#                         (*rhs_root.as_ptr()).p = Some(rmost);
#                     }
#                     (*rmost.as_ptr()).upd_subtree();
#                     self.size = (*rmost.as_ptr()).subt;
#                 }
#             }
#             rhs.root = None;
#             rhs.size = 0;
#         }
#         pub fn merge_left(&mut self, mut lhs: Self) {
#             if self.len() == 0 {
#                 self.root = lhs.root;
#                 self.size = lhs.size;
#             } else {
#                 unsafe {
#                     let lmost = self.kth_ptr(0).unwrap();
#                     self.splay(lmost);
#                     (*lmost.as_ptr()).l = lhs.root;
#                     if let Some(lhs_root) = lhs.root {
#                         (*lhs_root.as_ptr()).p = Some(lmost);
#                     }
#                     (*lmost.as_ptr()).upd_subtree();
#                     self.size = (*lmost.as_ptr()).subt;
#                 }
#             }
#             lhs.root = None;
#             lhs.size = 0;
#         }
#     }
#     impl<T: Debug> Debug for Rope<T> {
#         fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
#             write!(f, "[")?;
#             let mut cnt: usize = 0;
#             unsafe {
#                 let mut stack: Vec<*mut Node<T>> = Vec::new();
#                 let mut curr = self.root;
#                 loop {
#                     while let Some(x) = curr {
#                         stack.push(x.as_ptr());
#                         curr = (*x.as_ptr()).l;
#                     }
#                     if let Some(x) = stack.pop() {
#                         if cnt == 0 {
#                             write!(f, "{:?}", (*x).data)?;
#                         } else {
#                             write!(f, ", {:?}", (*x).data)?;
#                         }
#                         cnt += 1;
#                         curr = (*x).r;
#                     } else {
#                         break;
#                     }
#                 }
#             }
#             write!(f, "]")
#         }
#     }
#     impl<T: Display> Display for Rope<T> {
#         fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
#             unsafe {
#                 let mut stack: Vec<*mut Node<T>> = Vec::new();
#                 let mut curr = self.root;
#                 loop {
#                     while let Some(x) = curr {
#                         stack.push(x.as_ptr());
#                         curr = (*x.as_ptr()).l;
#                     }
#                     if let Some(x) = stack.pop() {
#                         write!(f, "{}", (*x).data)?;
#                         curr = (*x).r;
#                     } else {
#                         break;
#                     }
#                 }
#             }
#             Ok(())
#         }
#     }
#     impl<T> Drop for Rope<T> {
#         fn drop(&mut self) {
#             if let Some(root) = self.root {
#                 unsafe {
#                     let mut st: Vec<*mut Node<T>> = Vec::new();
#                     st.push(root.as_ptr());
#                     while let Some(t) = st.pop() {
#                         let v = Box::from_raw(t);
#                         if let Some(l) = v.l {
#                             st.push(l.as_ptr());
#                         }
#                         if let Some(r) = v.r {
#                             st.push(r.as_ptr());
#                         }
#                         // retrieve.drop()
#                     }
#                 }
#             }
#         }
#     }
#     impl<T> Index<usize> for Rope<T> {
#         type Output = T;
#         fn index(&self, idx: usize) -> &Self::Output {
#             unsafe {
#                 let p = self.kth_ptr(idx);
#                 &(*p.unwrap().as_ptr()).data
#             }
#         }
#     }
#     impl<T> IndexMut<usize> for Rope<T> {
#         fn index_mut(&mut self, idx: usize) -> &mut Self::Output {
#             unsafe {
#                 let p = self.kth_ptr(idx);
#                 &mut (*p.unwrap().as_ptr()).data
#             }
#         }
#     }
#     impl<T> FromIterator<T> for Rope<T> {
#         fn from_iter<I: IntoIterator<Item = T>>(iter: I) -> Self {
#             let mut arr = Self::new();
#             for v in iter {
#                 unsafe { arr.push_ontop_root(v) };
#             }
#             arr
#         }
#     }
#     //------------------------
#     // Helper implementations
#     //------------------------
#     impl<T> Rope<T> {
#         /// Adds data as a new root of a rope, and putting the original root
#         /// as a left child of the root.
#         unsafe fn push_ontop_root(&mut self, data: T) {
#             let new_node = NonNull::new_unchecked(Box::into_raw(Box::new(Node::new(data))));
#             if let Some(root) = self.root {
#                 (*root.as_ptr()).p = Some(new_node);
#                 (*new_node.as_ptr()).l = Some(root);
#             }
#             self.root = Some(new_node);
#             (*new_node.as_ptr()).upd_subtree();
#             self.size += 1;
#         }
#         /// Returns false if x has no parent, and do nothing
#         /// Returns true if x has a parent, after performing rotation
#         unsafe fn rotate(&mut self, x: NonNull<Node<T>>) -> bool {
#             if let Some((is_x_left, p)) = Node::is_left_child(x) {
#                 // Check if p is root
#                 if let Some(root) = self.root {
#                     if ptr::eq(root.as_ptr(), p.as_ptr()) {
#                         self.root = Some(x);
#                     }
#                 }
#                 // Connect x to xpp. If pp is None, do nothing.
#                 (*x.as_ptr()).p = (*p.as_ptr()).p;
#                 if let Some((is_p_left, pp)) = Node::is_left_child(p) {
#                     if is_p_left {
#                         (*pp.as_ptr()).l = Some(x);
#                     } else {
#                         (*pp.as_ptr()).r = Some(x);
#                     }
#                 }
#                 if is_x_left {
#                     let b = (*x.as_ptr()).r;
#                     (*x.as_ptr()).r = Some(p);
#                     (*p.as_ptr()).p = Some(x);
#                     (*p.as_ptr()).l = b;
#                     if let Some(b) = b {
#                         (*b.as_ptr()).p = Some(p);
#                     }
#                 } else {
#                     let b = (*x.as_ptr()).l;
#                     (*x.as_ptr()).l = Some(p);
#                     (*p.as_ptr()).p = Some(x);
#                     (*p.as_ptr()).r = b;
#                     if let Some(b) = b {
#                         (*b.as_ptr()).p = Some(p);
#                     }
#                 }
#                 (*p.as_ptr()).upd_subtree();
#                 (*x.as_ptr()).upd_subtree();
#                 true
#             } else {
#                 false
#             }
#         }
#         fn splay(&mut self, x: NonNull<Node<T>>) {
#             unsafe {
#                 while let Some(root) = self.root {
#                     if ptr::eq(x.as_ptr(), root.as_ptr()) {
#                         break;
#                     }
#                     if let Some((is_x_left, p)) = Node::is_left_child(x) {
#                         if ptr::eq(root.as_ptr(), p.as_ptr()) {
#                             // If p is root, rotate x once
#                             self.rotate(x);
#                         } else {
#                             // Panics if pp doesn't exist, which happens only when p is root
#                             let (is_p_left, _pp) = Node::is_left_child(p).unwrap();
#                             if is_x_left == is_p_left {
#                                 self.rotate(p);
#                                 self.rotate(x);
#                             } else {
#                                 self.rotate(x);
#                                 self.rotate(x);
#                             }
#                         }
#                     } else {
#                         // x has no parent, which should logically never happen
#                         unreachable!()
#                     }
#                 }
#             }
#         }
#         unsafe fn kth_ptr(&self, idx: usize) -> Link<T> {
#             if self.size <= idx {
#                 return None;
#             }
#             if let Some(r) = self.root {
#                 let mut rem = idx;
#                 let mut p = r;
#                 loop {
#                     let lsize = (*p.as_ptr()).left_size();
#                     match rem.cmp(&lsize) {
#                         Ordering::Less => {
#                             p = (*p.as_ptr()).l?;
#                         }
#                         Ordering::Equal => {
#                             break;
#                         }
#                         Ordering::Greater => {
#                             rem -= lsize + 1;
#                             p = (*p.as_ptr()).r?;
#                         }
#                     }
#                 }
#                 Some(p)
#             } else {
#                 None
#             }
#         }
#         unsafe fn remove_helper(&mut self, x: NonNull<Node<T>>) -> NonNull<Node<T>> {
#             // Set remove_target to the actual node to delete
#             match ((*x.as_ptr()).l, ((*x.as_ptr()).r)) {
#                 (None, None) => {
#                     // Reset root if the node itself is root
#                     if let Some(root) = self.root {
#                         if ptr::eq(root.as_ptr(), x.as_ptr()) {
#                             self.root = None;
#                         }
#                     }
#                     // Detatch itself from parent
#                     if let Some((is_x_left, p)) = Node::is_left_child(x) {
#                         if is_x_left {
#                             (*p.as_ptr()).l = None;
#                         } else {
#                             (*p.as_ptr()).r = None;
#                         }
#                         // Update subtree size
#                         let mut p = p;
#                         (*p.as_ptr()).upd_subtree();
#                         while let Some(pp) = (*p.as_ptr()).p {
#                             p = pp;
#                             (*p.as_ptr()).upd_subtree();
#                         }
#                     }
#                     x
#                 }
#                 (Some(l), None) => {
#                     // Reset root if the node itself is a root
#                     if let Some(root) = self.root {
#                         if ptr::eq(root.as_ptr(), x.as_ptr()) {
#                             self.root = Some(l);
#                         }
#                     }
#                     (*l.as_ptr()).p = (*x.as_ptr()).p;
#                     if let Some((is_rt_left, p)) = Node::is_left_child(x) {
#                         if is_rt_left {
#                             (*p.as_ptr()).l = Some(l);
#                         } else {
#                             (*p.as_ptr()).r = Some(l);
#                         }
#                     }
#                     let mut p = l;
#                     while let Some(pp) = (*p.as_ptr()).p {
#                         p = pp;
#                         (*p.as_ptr()).upd_subtree();
#                     }
#                     x
#                 }
#                 (None, Some(r)) => {
#                     // Reset root if the node itself is a root
#                     if let Some(root) = self.root {
#                         if ptr::eq(root.as_ptr(), x.as_ptr()) {
#                             self.root = Some(r);
#                         }
#                     }
#                     (*r.as_ptr()).p = (*x.as_ptr()).p;
#                     if let Some((is_rt_left, p)) = Node::is_left_child(x) {
#                         if is_rt_left {
#                             (*p.as_ptr()).l = Some(r);
#                         } else {
#                             (*p.as_ptr()).r = Some(r);
#                         }
#                     }
#                     let mut p = r;
#                     while let Some(pp) = (*p.as_ptr()).p {
#                         p = pp;
#                         (*p.as_ptr()).upd_subtree();
#                     }
#                     x
#                 }
#                 (Some(l), Some(_)) => {
#                     let mut sw = l;
#                     while let Some(sr) = (*sw.as_ptr()).r {
#                         sw = sr;
#                     }
#                     std::mem::swap(&mut (*x.as_ptr()).data, &mut (*sw.as_ptr()).data);
#                     sw = self.remove_helper(sw);
#                     sw
#                 }
#             }
#         }
#     }
#     //-----------
#     // Iterators
#     //-----------
#     impl<T> Rope<T> {
#         pub fn iter(&self) -> Iter<T> {
#             Iter::new(self)
#         }
#         pub fn iter_mut(&mut self) -> IterMut<T> {
#             IterMut::new(self)
#         }
#     }
#     pub struct Iter<'a, T> {
#         rope: &'a Rope<T>,
#         stack: Vec<NonNull<Node<T>>>,
#         curr: Link<T>,
#     }
#     impl<'a, T> Iter<'a, T> {
#         fn new(rope: &'a Rope<T>) -> Self {
#             let root = rope.root;
#             Self {
#                 rope,
#                 stack: Vec::new(),
#                 curr: root,
#             }
#         }
#     }
#     impl<'a, T> IntoIterator for &'a Rope<T> {
#         type Item = &'a T;
#         type IntoIter = Iter<'a, T>;
#         fn into_iter(self) -> Self::IntoIter {
#             Self::IntoIter::new(self)
#         }
#     }
#     impl<'a, T> Iterator for Iter<'a, T> {
#         type Item = &'a T;
#         fn next(&mut self) -> Option<Self::Item> {
#             unsafe {
#                 while let Some(x) = self.curr {
#                     self.stack.push(x);
#                     self.curr = (*x.as_ptr()).l;
#                 }
#                 if let Some(x) = self.stack.pop() {
#                     self.curr = (*x.as_ptr()).r;
#                     Some(&x.as_ref().data)
#                 } else {
#                     None
#                 }
#             }
#         }
#         fn size_hint(&self) -> (usize, Option<usize>) {
#             (self.rope.len(), Some(self.rope.len()))
#         }
#     }
#     pub struct IterMut<'a, T> {
#         rope: &'a mut Rope<T>,
#         stack: Vec<NonNull<Node<T>>>,
#         curr: Link<T>,
#     }
#     impl<'a, T> IterMut<'a, T> {
#         fn new(rope: &'a mut Rope<T>) -> Self {
#             let root = rope.root;
#             Self {
#                 rope,
#                 stack: Vec::new(),
#                 curr: root,
#             }
#         }
#     }
#     impl<'a, T> IntoIterator for &'a mut Rope<T> {
#         type Item = &'a mut T;
#         type IntoIter = IterMut<'a, T>;
#         fn into_iter(self) -> Self::IntoIter {
#             Self::IntoIter::new(self)
#         }
#     }
#     impl<'a, T> Iterator for IterMut<'a, T> {
#         type Item = &'a mut T;
#         fn next(&mut self) -> Option<Self::Item> {
#             unsafe {
#                 while let Some(x) = self.curr {
#                     self.stack.push(x);
#                     self.curr = (*x.as_ptr()).l;
#                 }
#                 if let Some(mut x) = self.stack.pop() {
#                     self.curr = (*x.as_ptr()).r;
#                     Some(&mut x.as_mut().data)
#                 } else {
#                     None
#                 }
#             }
#         }
#         fn size_hint(&self) -> (usize, Option<usize>) {
#             (self.rope.len(), Some(self.rope.len()))
#         }
#     }
# }
```

## Code
```rust,noplayground
mod rope {
    use std::{
        cmp::Ordering,
        fmt::{Debug, Display},
        ops::{Bound::*, Index, IndexMut, RangeBounds},
        ptr::{self, NonNull},
    };

    pub struct Rope<T> {
        root: Link<T>,
        size: usize,
    }

    impl<T> Default for Rope<T> {
        fn default() -> Self {
            Self {
                root: None,
                size: 0,
            }
        }
    }

    impl<T> Rope<T> {
        pub fn new() -> Self {
            Self::default()
        }

        pub fn len(&self) -> usize {
            self.size
        }

        pub fn clear(&mut self) {
            let drop_tree = Self {
                root: self.root,
                size: self.size,
            };
            drop(drop_tree);
            self.root = None;
            self.size = 0;
        }

        pub fn insert(&mut self, idx: usize, data: T) {
            debug_assert!(idx <= self.size);
            unsafe {
                let new_node = NonNull::new_unchecked(Box::into_raw(Box::new(Node::new(data))));

                if let Some(r) = self.root {
                    let idx = self.kth_ptr(idx);
                    if let Some(idx) = idx {
                        // idx_node is the node which new_node should replace
                        // "Replace" means the new_node should be placed right before the idx_node
                        if let Some(l) = (*idx.as_ptr()).l {
                            // Attach at the right of rightmost node from l
                            let mut p = l;
                            while let Some(r) = (*p.as_ptr()).r {
                                p = r;
                            }
                            // Attach new_node to the right of p
                            (*new_node.as_ptr()).p = Some(p);
                            (*p.as_ptr()).r = Some(new_node);
                        } else {
                            // Attach it right away
                            let p = idx;
                            (*new_node.as_ptr()).p = Some(p);
                            (*p.as_ptr()).l = Some(new_node);
                        }
                    } else {
                        // idx == self.size
                        // new_node goes to the rightmost of the tree
                        let mut p = r;
                        while let Some(r) = (*p.as_ptr()).r {
                            p = r;
                        }
                        // Attach new_node to the right of p
                        (*new_node.as_ptr()).p = Some(p);
                        (*p.as_ptr()).r = Some(new_node);
                    }

                    let mut c = new_node;
                    while let Some(p) = (*c.as_ptr()).p {
                        c = p;
                        (*c.as_ptr()).upd_subtree();
                    }
                } else {
                    self.root = Some(new_node);
                }

                self.splay(new_node);
                self.size += 1;
            }
        }

        pub fn remove(&mut self, idx: usize) -> Option<T> {
            if idx >= self.size {
                return None;
            }

            let data: T = unsafe {
                if let Some(mut rt) = self.kth_ptr(idx) {
                    rt = self.remove_helper(rt);
                    if let Some(rp) = (*rt.as_ptr()).p {
                        self.splay(rp);
                    }
                    let retr = Box::from_raw(rt.as_ptr());
                    retr.data
                } else {
                    unreachable!()
                }
            };

            self.size -= 1;
            Some(data)
        }

        pub fn push_front(&mut self, data: T) {
            self.insert(0, data);
        }

        pub fn push_back(&mut self, data: T) {
            self.insert(self.size, data);
        }

        pub fn pop_front(&mut self) -> Option<T> {
            self.remove(0)
        }

        pub fn pop_back(&mut self) -> Option<T> {
            self.remove(self.size - 1)
        }

        /// Splits out the rope, leaving self[..at] and returning self[at..].
        /// If the index is invalid, it returns None.
        pub fn take_right(&mut self, right_start: usize) -> Option<Self> {
            let rhs = unsafe {
                if right_start == 0 {
                    let rhs = Self {
                        root: self.root,
                        size: self.size,
                    };
                    self.root = None;
                    self.size = 0;
                    rhs
                } else {
                    let root = self.kth_ptr(right_start - 1)?;
                    self.splay(root);
                    if let Some(r) = (*root.as_ptr()).r {
                        (*root.as_ptr()).r = None;
                        (*r.as_ptr()).p = None;
                        (*root.as_ptr()).upd_subtree();
                        self.size = (*root.as_ptr()).subt;
                        Self {
                            root: Some(r),
                            size: (*r.as_ptr()).subt,
                        }
                    } else {
                        Self {
                            root: None,
                            size: 0,
                        }
                    }
                }
            };
            Some(rhs)
        }

        /// Splits out the rope and returns self[..at] and self[at..].
        /// If the index is invalid, it returns None.
        pub fn split_at(mut self, at: usize) -> Option<(Self, Self)> {
            let rhs = self.take_right(at)?;
            Some((self, rhs))
        }

        /// Takes out the range from the rope.
        /// Returns None if the index is invalid.
        pub fn take_range(&mut self, range: impl RangeBounds<usize>) -> Option<Self> {
            let l = match range.start_bound() {
                Included(&l) => l,
                Excluded(&l) => l + 1,
                Unbounded => 0,
            };
            let r = match range.end_bound() {
                Included(&r) => r + 1,
                Excluded(&r) => r,
                Unbounded => self.size,
            };

            if l > r || l > self.size || r > self.size {
                return None;
            }
            // Now the operations below never ends early
            let c = self.take_right(r)?;
            let b = self.take_right(l)?;
            self.merge_right(c);
            Some(b)
        }

        pub fn merge_right(&mut self, mut rhs: Self) {
            if self.len() == 0 {
                self.root = rhs.root;
                self.size = rhs.size;
            } else {
                unsafe {
                    let rmost = self.kth_ptr(self.size - 1).unwrap();
                    self.splay(rmost);
                    (*rmost.as_ptr()).r = rhs.root;
                    if let Some(rhs_root) = rhs.root {
                        (*rhs_root.as_ptr()).p = Some(rmost);
                    }
                    (*rmost.as_ptr()).upd_subtree();
                    self.size = (*rmost.as_ptr()).subt;
                }
            }
            rhs.root = None;
            rhs.size = 0;
        }

        pub fn merge_left(&mut self, mut lhs: Self) {
            if self.len() == 0 {
                self.root = lhs.root;
                self.size = lhs.size;
            } else {
                unsafe {
                    let lmost = self.kth_ptr(0).unwrap();
                    self.splay(lmost);
                    (*lmost.as_ptr()).l = lhs.root;
                    if let Some(lhs_root) = lhs.root {
                        (*lhs_root.as_ptr()).p = Some(lmost);
                    }
                    (*lmost.as_ptr()).upd_subtree();
                    self.size = (*lmost.as_ptr()).subt;
                }
            }
            lhs.root = None;
            lhs.size = 0;
        }

        /// Inserts rope into self at self.
        /// After the operation, rope[0] becomes self[at].
        /// Returns false if the specified index is invalid, true otherwise.
        pub fn insert_rope(&mut self, rope: Self, at: usize) -> bool {
            let rhs = self.take_right(at);
            if let Some(rhs) = rhs {
                self.merge_right(rope);
                self.merge_right(rhs);
                true
            } else {
                false
            }
        }
    }

    impl<T: Clone> Clone for Rope<T> {
        fn clone(&self) -> Self {
            self.iter().cloned().collect()
        }
    }

    impl<T: Debug> Debug for Rope<T> {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
            write!(f, "[")?;
            let mut cnt: usize = 0;
            unsafe {
                let mut stack: Vec<*mut Node<T>> = Vec::new();
                let mut curr = self.root;
                loop {
                    while let Some(x) = curr {
                        stack.push(x.as_ptr());
                        curr = (*x.as_ptr()).l;
                    }
                    if let Some(x) = stack.pop() {
                        if cnt == 0 {
                            write!(f, "{:?}", (*x).data)?;
                        } else {
                            write!(f, ", {:?}", (*x).data)?;
                        }
                        cnt += 1;
                        curr = (*x).r;
                    } else {
                        break;
                    }
                }
            }
            write!(f, "]")
        }
    }

    impl<T: Display> Display for Rope<T> {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
            unsafe {
                let mut stack: Vec<*mut Node<T>> = Vec::new();
                let mut curr = self.root;
                loop {
                    while let Some(x) = curr {
                        stack.push(x.as_ptr());
                        curr = (*x.as_ptr()).l;
                    }
                    if let Some(x) = stack.pop() {
                        write!(f, "{}", (*x).data)?;
                        curr = (*x).r;
                    } else {
                        break;
                    }
                }
            }
            Ok(())
        }
    }

    impl<T> Drop for Rope<T> {
        fn drop(&mut self) {
            if let Some(root) = self.root {
                unsafe {
                    let mut st: Vec<*mut Node<T>> = Vec::new();
                    st.push(root.as_ptr());
                    while let Some(t) = st.pop() {
                        let v = Box::from_raw(t);
                        if let Some(l) = v.l {
                            st.push(l.as_ptr());
                        }
                        if let Some(r) = v.r {
                            st.push(r.as_ptr());
                        }
                        drop(v);
                    }
                }
            }
        }
    }

    impl<T> Index<usize> for Rope<T> {
        type Output = T;
        fn index(&self, idx: usize) -> &Self::Output {
            unsafe {
                let p = self.kth_ptr(idx).unwrap();
                &(*p.as_ptr()).data
            }
        }
    }

    impl<T> IndexMut<usize> for Rope<T> {
        fn index_mut(&mut self, idx: usize) -> &mut Self::Output {
            unsafe {
                let p = self.kth_ptr(idx).unwrap();
                self.splay(p);
                &mut (*p.as_ptr()).data
            }
        }
    }

    impl<T> FromIterator<T> for Rope<T> {
        fn from_iter<I: IntoIterator<Item = T>>(iter: I) -> Self {
            let mut arr = Self::new();
            for v in iter {
                unsafe { arr.push_ontop_root(v) };
            }
            arr
        }
    }

    impl<T> Rope<T> {
        pub fn iter(&self) -> Iter<T> {
            Iter::new(self)
        }

        pub fn iter_mut(&mut self) -> IterMut<T> {
            IterMut::new(self)
        }
    }

    pub struct Iter<'a, T> {
        rope: &'a Rope<T>,
        stack: Vec<NonNull<Node<T>>>,
        curr: Link<T>,
    }

    impl<'a, T> Iter<'a, T> {
        fn new(rope: &'a Rope<T>) -> Self {
            let root = rope.root;
            Self {
                rope,
                stack: Vec::new(),
                curr: root,
            }
        }
    }

    impl<'a, T> IntoIterator for &'a Rope<T> {
        type Item = &'a T;
        type IntoIter = Iter<'a, T>;
        fn into_iter(self) -> Self::IntoIter {
            Self::IntoIter::new(self)
        }
    }

    impl<'a, T> Iterator for Iter<'a, T> {
        type Item = &'a T;
        fn next(&mut self) -> Option<Self::Item> {
            unsafe {
                while let Some(x) = self.curr {
                    self.stack.push(x);
                    self.curr = (*x.as_ptr()).l;
                }
                if let Some(x) = self.stack.pop() {
                    self.curr = (*x.as_ptr()).r;
                    Some(&x.as_ref().data)
                } else {
                    None
                }
            }
        }

        fn size_hint(&self) -> (usize, Option<usize>) {
            (self.rope.len(), Some(self.rope.len()))
        }
    }

    pub struct IterMut<'a, T> {
        rope: &'a mut Rope<T>,
        stack: Vec<NonNull<Node<T>>>,
        curr: Link<T>,
    }

    impl<'a, T> IterMut<'a, T> {
        fn new(rope: &'a mut Rope<T>) -> Self {
            let root = rope.root;
            Self {
                rope,
                stack: Vec::new(),
                curr: root,
            }
        }
    }

    impl<'a, T> IntoIterator for &'a mut Rope<T> {
        type Item = &'a mut T;
        type IntoIter = IterMut<'a, T>;
        fn into_iter(self) -> Self::IntoIter {
            Self::IntoIter::new(self)
        }
    }

    impl<'a, T> Iterator for IterMut<'a, T> {
        type Item = &'a mut T;
        fn next(&mut self) -> Option<Self::Item> {
            unsafe {
                while let Some(x) = self.curr {
                    self.stack.push(x);
                    self.curr = (*x.as_ptr()).l;
                }
                if let Some(mut x) = self.stack.pop() {
                    self.curr = (*x.as_ptr()).r;
                    Some(&mut x.as_mut().data)
                } else {
                    None
                }
            }
        }

        fn size_hint(&self) -> (usize, Option<usize>) {
            (self.rope.len(), Some(self.rope.len()))
        }
    }

    //------------------------
    // Helper implementations
    //------------------------

    struct Node<T> {
        data: T,
        subt: usize,
        l: Link<T>,
        r: Link<T>,
        p: Link<T>,
    }

    type Link<T> = Option<NonNull<Node<T>>>;

    impl<T> Node<T> {
        fn new(data: T) -> Self {
            Node {
                data,
                subt: 1,
                l: None,
                r: None,
                p: None,
            }
        }
        fn left_size(&self) -> usize {
            unsafe { self.l.map_or(0, |l| (*l.as_ptr()).subt) }
        }
        fn right_size(&self) -> usize {
            unsafe { self.r.map_or(0, |r| (*r.as_ptr()).subt) }
        }
        fn upd_subtree(&mut self) {
            self.subt = 1 + self.left_size() + self.right_size();
        }

        // Option<(is_left, parent)>
        unsafe fn is_left_child(x: NonNull<Self>) -> Option<(bool, NonNull<Self>)> {
            if let Some(p) = (*x.as_ptr()).p {
                if (*p.as_ptr())
                    .l
                    .map_or(false, |pl| ptr::eq(x.as_ptr(), pl.as_ptr()))
                {
                    Some((true, p))
                } else {
                    Some((false, p))
                }
            } else {
                None
            }
        }
    }

    impl<T> Rope<T> {
        /// Adds data as a new root of a rope, and putting the original root
        /// as a left child of the root.
        unsafe fn push_ontop_root(&mut self, data: T) {
            let new_node = NonNull::new_unchecked(Box::into_raw(Box::new(Node::new(data))));
            if let Some(root) = self.root {
                (*root.as_ptr()).p = Some(new_node);
                (*new_node.as_ptr()).l = Some(root);
            }
            self.root = Some(new_node);
            (*new_node.as_ptr()).upd_subtree();
            self.size += 1;
        }

        /// Returns false if x has no parent, and do nothing
        /// Returns true if x has a parent, after performing rotation
        unsafe fn rotate(&mut self, x: NonNull<Node<T>>) -> bool {
            if let Some((is_x_left, p)) = Node::is_left_child(x) {
                // Check if p is root
                if let Some(root) = self.root {
                    if ptr::eq(root.as_ptr(), p.as_ptr()) {
                        self.root = Some(x);
                    }
                }

                // Connect x to xpp. If pp is None, do nothing.
                (*x.as_ptr()).p = (*p.as_ptr()).p;
                if let Some((is_p_left, pp)) = Node::is_left_child(p) {
                    if is_p_left {
                        (*pp.as_ptr()).l = Some(x);
                    } else {
                        (*pp.as_ptr()).r = Some(x);
                    }
                }

                if is_x_left {
                    let b = (*x.as_ptr()).r;
                    (*x.as_ptr()).r = Some(p);
                    (*p.as_ptr()).p = Some(x);
                    (*p.as_ptr()).l = b;
                    if let Some(b) = b {
                        (*b.as_ptr()).p = Some(p);
                    }
                } else {
                    let b = (*x.as_ptr()).l;
                    (*x.as_ptr()).l = Some(p);
                    (*p.as_ptr()).p = Some(x);
                    (*p.as_ptr()).r = b;
                    if let Some(b) = b {
                        (*b.as_ptr()).p = Some(p);
                    }
                }

                (*p.as_ptr()).upd_subtree();
                (*x.as_ptr()).upd_subtree();
                true
            } else {
                false
            }
        }

        fn splay(&mut self, x: NonNull<Node<T>>) {
            unsafe {
                while let Some(root) = self.root {
                    if ptr::eq(x.as_ptr(), root.as_ptr()) {
                        break;
                    }

                    if let Some((is_x_left, p)) = Node::is_left_child(x) {
                        if ptr::eq(root.as_ptr(), p.as_ptr()) {
                            // If p is root, rotate x once
                            self.rotate(x);
                        } else {
                            // Panics if pp doesn't exist, which happens only when p is root
                            let (is_p_left, _pp) = Node::is_left_child(p).unwrap();
                            if is_x_left == is_p_left {
                                self.rotate(p);
                                self.rotate(x);
                            } else {
                                self.rotate(x);
                                self.rotate(x);
                            }
                        }
                    } else {
                        // x has no parent, which should logically never happen
                        unreachable!()
                    }
                }
            }
        }

        unsafe fn kth_ptr(&self, idx: usize) -> Link<T> {
            if self.size <= idx {
                return None;
            }
            if let Some(r) = self.root {
                let mut rem = idx;
                let mut p = r;
                loop {
                    let lsize = (*p.as_ptr()).left_size();
                    match rem.cmp(&lsize) {
                        Ordering::Less => {
                            p = (*p.as_ptr()).l?;
                        }
                        Ordering::Equal => {
                            break;
                        }
                        Ordering::Greater => {
                            rem -= lsize + 1;
                            p = (*p.as_ptr()).r?;
                        }
                    }
                }
                Some(p)
            } else {
                None
            }
        }

        unsafe fn remove_helper(&mut self, x: NonNull<Node<T>>) -> NonNull<Node<T>> {
            // Set remove_target to the actual node to delete
            match ((*x.as_ptr()).l, ((*x.as_ptr()).r)) {
                (None, None) => {
                    // Reset root if the node itself is root
                    if let Some(root) = self.root {
                        if ptr::eq(root.as_ptr(), x.as_ptr()) {
                            self.root = None;
                        }
                    }
                    // Detatch itself from parent
                    if let Some((is_x_left, p)) = Node::is_left_child(x) {
                        if is_x_left {
                            (*p.as_ptr()).l = None;
                        } else {
                            (*p.as_ptr()).r = None;
                        }
                        // Update subtree size
                        let mut p = p;
                        (*p.as_ptr()).upd_subtree();
                        while let Some(pp) = (*p.as_ptr()).p {
                            p = pp;
                            (*p.as_ptr()).upd_subtree();
                        }
                    }
                    x
                }
                (Some(l), None) => {
                    // Reset root if the node itself is a root
                    if let Some(root) = self.root {
                        if ptr::eq(root.as_ptr(), x.as_ptr()) {
                            self.root = Some(l);
                        }
                    }

                    (*l.as_ptr()).p = (*x.as_ptr()).p;
                    if let Some((is_rt_left, p)) = Node::is_left_child(x) {
                        if is_rt_left {
                            (*p.as_ptr()).l = Some(l);
                        } else {
                            (*p.as_ptr()).r = Some(l);
                        }
                    }

                    let mut p = l;
                    while let Some(pp) = (*p.as_ptr()).p {
                        p = pp;
                        (*p.as_ptr()).upd_subtree();
                    }
                    x
                }
                (None, Some(r)) => {
                    // Reset root if the node itself is a root
                    if let Some(root) = self.root {
                        if ptr::eq(root.as_ptr(), x.as_ptr()) {
                            self.root = Some(r);
                        }
                    }

                    (*r.as_ptr()).p = (*x.as_ptr()).p;
                    if let Some((is_rt_left, p)) = Node::is_left_child(x) {
                        if is_rt_left {
                            (*p.as_ptr()).l = Some(r);
                        } else {
                            (*p.as_ptr()).r = Some(r);
                        }
                    }

                    let mut p = r;
                    while let Some(pp) = (*p.as_ptr()).p {
                        p = pp;
                        (*p.as_ptr()).upd_subtree();
                    }
                    x
                }
                (Some(l), Some(_)) => {
                    let mut sw = l;
                    while let Some(sr) = (*sw.as_ptr()).r {
                        sw = sr;
                    }
                    std::mem::swap(&mut (*x.as_ptr()).data, &mut (*sw.as_ptr()).data);
                    sw = self.remove_helper(sw);
                    sw
                }
            }
        }
    }
}
```
