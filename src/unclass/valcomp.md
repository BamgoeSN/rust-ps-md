# Value Compression

## w/ HashMap
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

## w/0 HashMap
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