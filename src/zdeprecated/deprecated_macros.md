# Deprecated Macros

## HashMap

Deprecated because it's better to utilize the fact that `HashMap<K, V>` is `From<[(K, V); N]>` for data hardcoding of a hashmap.

```rust,noplayground
macro_rules! count_tts {
    () => { 0 };
    ($odd:tt $($a:tt $b:tt)*) => { (count_tts!($($a)*) << 1) | 1 };
    ($($a:tt $even:tt)*) => { count_tts!($($a)*) << 1 };
}

// let map: HashMap<i64, i64> = hashmap![1,1; 2,2; 3,3];
macro_rules! hashmap {
    ($($k:expr,$v:expr);*) => {{
        let mut map = HashMap::with_capacity(count_tts![$($k )*]);
        $( map.insert($k, $v); )*
        map
    }}
}
```

---

Last modified on 231008.