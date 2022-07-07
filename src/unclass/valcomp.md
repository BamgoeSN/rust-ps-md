# Value Compression

## Example

```rust
# fn main() {
let arr: Vec<i32> = vec![1, 2, 4, 7, 9, 7, 4, 2, 1];

let (compressor, reevaluator) = compress_value(&arr);

let compr: Vec<usize> = arr.iter().map(|x| *compressor.get(x).unwrap()).collect();
println!("{:?}", compr);    // [0, 1, 2, 3, 4, 3, 2, 1, 0]

let original: Vec<i32> = compr.iter().map(|&i| *reevaluator[i]).collect();
println!("{:?}", original); // [1, 2, 4, 7, 9, 7, 4, 2, 1]
# }
# 
# /// compressor[original_value] = compressed_value
# /// reevaluator[compressed_value] = original_value
# fn compress_value<T: Ord>(arr: &[T]) -> (std::collections::BTreeMap<&T, usize>, Vec<&T>) {
#     use std::collections::*;
#     let compressor: BTreeMap<&T, usize> = {
#         let mut sorted: Vec<_> = arr.iter().collect();
#         sorted.sort_unstable();
#         sorted.dedup();
#         sorted.into_iter().enumerate().map(|x| (x.1, x.0)).collect()
#     };
#     let reevaluator: Vec<&T> = compressor.iter().map(|x| *x.0).collect();
#     (compressor, reevaluator)
# }
```

## Code

### w/ HashMap
```rust,noplayground
/// compressor[original_value] = compressed_value
/// reevaluator[compressed_value] = original_value
fn compress_value<T: Ord + std::hash::Hash>(
    arr: &[T],
) -> (std::collections::HashMap<&T, usize>, Vec<&T>) {
    use std::collections::*;
    let compressor: HashMap<&T, usize> = {
        let mut sorted: Vec<_> = arr.iter().collect();
        sorted.sort_unstable();
        sorted.dedup();
        sorted.into_iter().enumerate().map(|x| (x.1, x.0)).collect()
    };
    let reevaluator: Vec<&T> = compressor.iter().map(|x| *x.0).collect();
    (compressor, reevaluator)
}
```

### w/o HashMap
```rust,noplayground
/// compressor[original_value] = compressed_value
/// reevaluator[compressed_value] = original_value
fn compress_value<T: Ord>(arr: &[T]) -> (std::collections::BTreeMap<&T, usize>, Vec<&T>) {
    use std::collections::*;
    let compressor: BTreeMap<&T, usize> = {
        let mut sorted: Vec<_> = arr.iter().collect();
        sorted.sort_unstable();
        sorted.dedup();
        sorted.into_iter().enumerate().map(|x| (x.1, x.0)).collect()
    };
    let reevaluator: Vec<&T> = compressor.iter().map(|x| *x.0).collect();
    (compressor, reevaluator)
}
```