# 题目

https://leetcode-cn.com/problems/kth-largest-element-in-an-array/submissions/

# 使用堆排序的方式

```java
package com.ymj.demo.algorithm.array;

import java.util.Random;

/**
 * 数组中的第K个最大元素
 */
public class FindKthLargest {

    public int findKthLargest(int[] nums, int k) {
        return findKthLargestByFast(nums, k);
    }

    /**
     * 使用堆排序找出第K大的节点
     *
     * @return
     */
    private int findKthLargestByHeap(int[] nums, int k) {
        // 建立最大堆
        buildHeap(nums);
        // 每次将堆中最大的值移动到数组的末尾，然后将堆的大小缩小1，找寻K次即可完成
        int target = 0;
        int endIndex = nums.length - 1;
        for (int i = 0; i < k; i++) {
            target = nums[0];
            // 数据交换
            nums[0] = nums[endIndex];
            // 下沉堆，堆大小缩小到endIndex
            siftDown(nums, 0, nums[endIndex], endIndex--);
        }
        return target;
    }

    private void buildHeap(int[] nums) {
        /**
         * 根据完全二叉树的性质：非叶子节点 n2+n1 = floor(n/2)
         * 也就是说只需要进行 floor(n/2) 次下沉（将所有非叶子节点下沉可可构建成一个堆）
         */
        for (int i = nums.length >>> 1; i >= 0; i--) {
            siftDown(nums, i, nums[i], nums.length);
        }
    }

    private void siftDown(int[] nums, int i, int num, int length) {
        // 下沉到叶子节点最多只会到索引大雨 n2+n1 = floor(n/2)
        int half = length >>> 1;
        while (i < half) {
            // 先找左孩子,右孩子
            int child = (i << 1) + 1;// 由于i < half 所以 i*2+1 是小于length的所以不需要作越界检查
            int rightChild = child + 1;
            // 找到其中最大的值
            int c = nums[child];
            if (rightChild < length && nums[rightChild] > c) {
                c = nums[child = rightChild];
            }
            if (c < num) {
                break;
            }
            nums[i] = c;
            i = child;
        }
        // 到叶子节点了
        nums[i] = num;
    }

    public int findKthLargest2(int[] nums, int k) {
        return findKthLargestByFast(nums, k);
    }

    /**
     * 快排的方式
     * @param nums
     * @param k
     * @return
     */
    private int findKthLargestByFast(int[] nums, int k) {
        // 目标索引
        int targetIndex = nums.length - k;
        int point = findPoint(nums, 0, nums.length);
        if (point == targetIndex) {
            return nums[point];
        }
        return point < targetIndex ? findPoint(nums, point + 1, nums.length) : findPoint(nums, 0, point - 1);
    }

    private int findPoint(int[] nums, int s, int e) {
        Random random = new Random();
        int i = s + random.nextInt(e - 1);
        swap(nums, s, i);
        return partition(nums, s, e);
    }

    private int partition(int[] nums, int s, int e) {
        int limit = nums[s];
        // 保证基准在早期不被移动
        int index = s + 1;
        for (int j = index; j < e; j++) {
            if (nums[j] < limit) {
                swap(nums, index, j);
                index++;
            }
        }
        /**
         * 由于index最后所指向的地方的数据都闭 limit 大，[0,limit)的数据都<= index ,
         * 所以只需要将 limit 和 index-1 交换位置即可
         */
        swap(nums, s, index - 1);
        return index - 1;
    }

    private void swap(int[] nums, int s, int i) {
        if (s != i) {
            nums[s] = nums[s] ^ nums[i];
            nums[i] = nums[s] ^ nums[i];
            nums[s] = nums[s] ^ nums[i];
        }
    }


    public static void main(String[] args) {
        int[] ints = {-1, 2, 0};
        System.out.println(new FindKthLargest().findKthLargest(ints, 3));
    }

}

```

