---
title: 计算边长200以内的直角三角形有那些？
categories:
  - algorithm
tags: algorithm
date: 2024-04-29 11:44:51
---

# 计算边长200以内的直角三角形有那些？用python，java，go，rust，haskell，c，c++实现

Python:
```python
import math

def find_right_triangles(max_side):
    triangles = []
    for a in range(1, max_side + 1):
        for b in range(a, max_side + 1):
            c = int(math.sqrt(a**2 + b**2))
            if c <= max_side and a**2 + b**2 == c**2:
                triangles.append((a, b, c))
    return triangles

print(find_right_triangles(200))

```

Java:
```java
import java.util.ArrayList;
import java.util.List;

public class RightTriangles {
    public static List<int[]> findRightTriangles(int maxSide) {
        List<int[]> triangles = new ArrayList<>();
        for (int a = 1; a <= maxSide; a++) {
            for (int b = a; b <= maxSide; b++) {
                int c = (int) Math.sqrt(a * a + b * b);
                if (c <= maxSide && a * a + b * b == c * c) {
                    triangles.add(new int[] {a, b, c});
                }
            }
        }
        return triangles;
    }

    public static void main(String[] args) {
        System.out.println(findRightTriangles(200));
    }
}
```

Go:
```go
package main

import (
    "fmt"
    "math"
)

func findRightTriangles(maxSide int) [][3]int {
    var triangles [][3]int
    for a := 1; a <= maxSide; a++ {
        for b := a; b <= maxSide; b++ {
            c := int(math.Sqrt(float64(a*a + b*b)))
            if c <= maxSide && a*a+b*b == c*c {
                triangles = append(triangles, [3]int{a, b, c})
            }
        }
    }
    return triangles
}

func main() {
    fmt.Println(findRightTriangles(200))
}
```

Rust:
```rust
use std::collections::HashSet;

fn find_right_triangles(max_side: i32) -> HashSet<(i32, i32, i32)> {
    let mut triangles: HashSet<(i32, i32, i32)> = HashSet::new();
    for a in 1..=max_side {
        for b in a..=max_side {
            let c = (a * a + b * b).sqrt() as i32;
            if c <= max_side && a * a + b * b == c * c {
                triangles.insert((a, b, c));
            }
        }
    }
    triangles
}

fn main() {
    println!("{:?}", find_right_triangles(200));
}
```

Haskell:
```haskell
import Data.Set (fromList, toList)
import Data.Tuple (swap)
import Data.List (nub)
import Data.Function (on)
import Data.Ord (comparing)

findRightTriangles :: Int -> [(Int, Int, Int)]
findRightTriangles maxSide = nub . toList $ fromList [(a, b, c) | a <- [1..maxSide], b <- [a..maxSide], let c = round $ sqrt $ fromIntegral $ a^2 + b^2, c <= maxSide, a^2 + b^2 == c^2]

main = print $ findRightTriangles 200
```

C:
```c
#include <stdio.h>
#include <math.h>

void findRightTriangles(int maxSide, int (*triangles)[3], int *count) {
    *count = 0;
    for (int a = 1; a <= maxSide; a++) {
        for (int b = a; b <= maxSide; b++) {
            int c = (int)sqrt(a * a + b * b);
            if (c <= maxSide && a * a + b * b == c * c) {
                triangles[*count][0] = a;
                triangles[*count][1] = b;
                triangles[*count][2] = c;
                (*count)++;
            }
        }
    }
}

int main() {
    int triangles[1000][3];
    int count;
    findRightTriangles(200, triangles, &count);
    for (int i = 0; i < count; i++) {
        printf("(%d, %d, %d)\n", triangles[i][0], triangles[i][1], triangles[i][2]);
    }
    return 0;
}
```

C++:
```c++
#include <iostream>
#include <vector>
#include <cmath>
#include <set>
#include <algorithm>

std::vector<std::vector<int>> findRightTriangles(int maxSide) {
    std::set<std::vector<int>> triangles;
    for (int a = 1; a <= maxSide; a++) {
        for (int b = a; b <= maxSide; b++) {
            int c = std::sqrt(a * a + b * b);
            if (c <= maxSide && a * a + b * b == c * c) {
                triangles.insert({a, b, c});
            }
        }
    }
    return std::vector<std::vector<int>>(triangles.begin(), triangles.end());
}

int main() {
    auto triangles = findRightTriangles(200);
    for (const auto& triangle : triangles) {
        std::cout << "(" << triangle[0] << ", " << triangle[1] << ", " << triangle[2] << ")" << std::endl;
    }
    return 0;
}
```
