# Value Compression

```rust,noplayground
/// Returns (compressor, reevaluator)
/// compressor[original_value] = compressed_value
/// reevaluator[compressed_value] = original_value
fn compress_value<T: Clone + Ord + Hash>(arr: &Vec<T>) -> (HashMap<T, usize>, Vec<T>) {
    let mut sorted = arr.clone();
    sorted.sort_unstable();

    let mut compressor: HashMap<T, usize> = HashMap::new();
    let mut reevaluator: Vec<T> = Vec::new();

    let mut prev: Option<&T> = None;
    let mut cmpr = 0usize;
    for v in sorted.iter() {
        if let Some(prev) = prev {
            if prev == v {
                continue;
            }
        }
        prev = Some(v);
        compressor.insert(v.clone(), cmpr);
        reevaluator.push(v.clone());
        cmpr += 1;
    }

    (compressor, reevaluator)
}
```