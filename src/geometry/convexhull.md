# Convex Hull

## Graham Scan
This snippet excludes every points on vertices from the convex hull, and only includes points of both ends.

### Snippet
```rust,noplayground
#[derive(Clone, Debug, Eq, PartialEq)]
struct Point {
    x: i64,
    y: i64,
}

impl Point {
    fn add(&self, other: &Self) -> Self {
        Self {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
    fn sub(&self, other: &Self) -> Self {
        Self {
            x: self.x - other.x,
            y: self.y - other.y,
        }
    }
}

#[derive(Clone, Debug, Eq, PartialEq)]
enum Turn {
    CW,
    CCW,
    Zero,
}

fn get_turn(v1: &Point, v2: &Point) -> Turn {
    let prod = v1.x * v2.y - v1.y * v2.x;
    if prod > 0 {
        Turn::CCW
    } else if prod < 0 {
        Turn::CW
    } else {
        Turn::Zero
    }
}

fn compare_ccw(a: &Point, b: &Point) -> std::cmp::Ordering {
    use std::cmp::Ordering;
    if a == b {
        return Ordering::Equal;
    }
    if a.x == 0 && a.y == 0 {
        return Ordering::Less;
    } else if b.x == 0 && b.y == 0 {
        return Ordering::Greater;
    }

    let turn = get_turn(a, b);
    if turn == Turn::CCW {
        return Ordering::Less;
    } else if turn == Turn::CW {
        return Ordering::Greater;
    }

    let a_dist = a.x * a.x + a.y * a.y;
    let b_dist = b.x * b.x + b.y * b.y;
    a_dist.cmp(&b_dist)
}

fn convex_hull(arr: &mut [Point]) -> Vec<Point> {
    let pivot = {
        let mut pivot = Point {
            x: i64::MAX,
            y: i64::MAX,
        };
        for v in arr.iter() {
            if pivot.y > v.y || (pivot.y == v.y && pivot.x > v.x) {
                pivot.x = v.x;
                pivot.y = v.y;
            }
        }
        pivot
    };

    for v in arr.iter_mut() {
        *v = v.sub(&pivot);
    }

    arr.sort_unstable_by(compare_ccw);

    let mut stack = vec![arr[0].clone()];
    let mut i = 1;
    while i < arr.len() {
        if stack.len() == 1 {
            if arr[i] != stack[0] {
                stack.push(arr[i].clone());
            }
            i += 1;
            continue;
        }

        let a = &arr[i];
        let b = &stack[stack.len() - 1];
        let c = &stack[stack.len() - 2];
        match get_turn(&b.sub(c), &a.sub(b)) {
            Turn::CCW => {
                stack.push(a.clone());
                i += 1;
            }
            Turn::CW => {
                stack.pop();
            }
            _ => {
                stack.pop();
                stack.push(a.clone());
                i += 1;
            }
        }
    }

    for v in arr.iter_mut() {
        *v = v.add(&pivot);
    }
    for v in stack.iter_mut() {
        *v = v.add(&pivot);
    }

    stack
}
```