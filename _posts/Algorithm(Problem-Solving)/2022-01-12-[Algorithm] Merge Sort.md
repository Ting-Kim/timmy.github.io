---
layout: post
title: "[Algorithm] Merge Sort (병합정렬)"
categories: Algorithm(Problem-Solving)
tags: [algorithm, Merge Sort]
comments: true
---

{% raw %}

매번 기본 라이브러리의 sort 함수를 사용했었는데, 기본이 부족한 것 같아서 오랜만에 `Merge Sort`를 찾아보며 구현했다.

파이썬에서는 전역변수를 통해 편하게 사용하곤 했었는데, 코드의 퀄리티를 고민하다 보니 전역변수는 옳지 않았다.

**매개변수는 레지스터를 이용하기 때문에 빠르다** (레지스터 > L2캐시 > 메모리)

ARM 64비트 기준 매개변수 6개를 사용할 수 있다.

이보다 더 많이 사용하는 경우가 많을 수 있지 않나? 

-> **그래서 객체나 자료구조들이 존재하는 것!**

``` java
import java.util.Arrays;

public class MergeSort {

    public static void main(String[] args) {
        
        int[][] array = {{1, 5}, {7, 6}, {3, 4}, {5, 1}, {2, 9}, {8, 3}, {4, 8}, {2, 6}};
        mergeSort(array);
    }

    static void mergeSort(int[][] array) {
        int[][] newArray = new int[array.length][array[0].length];
        mergeSort(array, newArray, 0, newArray.length - 1);
        for (int[] tmp: array) {
            System.out.print(" " + Arrays.toString(tmp));
        }
    }

    static void mergeSort(int[][] array, int[][] newArray, int startIdx, int endIdx) {
        
        if (startIdx < endIdx) {
            int middle = (startIdx + endIdx) / 2;
            mergeSort(array, newArray, startIdx, middle);
            mergeSort(array, newArray, middle + 1, endIdx);
            merge(array, newArray, startIdx, middle, endIdx);
            
        } 
        
    }

    static void merge(int[][] array, int[][] newArray, int startIdx, int middle, int endIdx) {
        for (int i = startIdx; i <= endIdx; i++) {
            newArray[i] = array[i];
        }
        int part1 = startIdx;
        int part2 = middle + 1;
        int idx = startIdx;
        
        while (part1 <= middle && part2 <= endIdx) {
            if (newArray[part1][0] < newArray[part2][0]) {
                array[idx++] = newArray[part1++];
            } else {
                array[idx++] = newArray[part2++];
            }
        }
        
        if (part1 <= middle) {
            for (int i = part1; i <= middle; i++) {
                array[idx++] = newArray[i++];
            }
        }

    }     
    
}

```

{% endraw %}