## 获取N个元素中的前K个高频元素 

#### 题目描述：

给定一个非空的整数数组，返回其中出现频率前K高的元素

示例1：

输入：nums=[1,1,1,2,2,3],k=2

输出：[1,2]

示例2：

输入：nums=[1],k=1

输出:[1]

说明：

1.你可以假设给定的K总是合理的，且1<=k<=数组中不相同的元素个数

2.你的算法时间复杂度必须优于O(nlogn)，n是数组的大小。

#### 题目思路：

**1.遍历元素集，将具体的key值与其出现频率保存Map中;**

**2.使用优先级队列(最大堆)存储出现频率最高的前k个键值对;**

**3.使用List存储优先队列中的所有key**

#### 实现方法：

>双层循环
>
>最大堆
>
>最小堆
>
>桶排序

#### 1.双层循环(时间复杂度不满足题目要求，但也可以实现)

```java
package heapTask;

import java.util.*;

/**
 * 双层循环实现
 * 时间复杂度：O(nlogn)，其中n表示数组长度
 * 空间复杂度：O(n),最极端情况下(每个元素都不同)，用于存储元素及其频率的Map需要存储n个键值对
 */
public class Solution2 {
    public List<Integer> topKFrequent(Integer[] nums,int k){
        //第一步：统计元素频率存入map中
        Map<Integer,Integer> map=new HashMap<>();
        for(int i:nums){
            //若当前元素频率为空，说明当前元素为第一次出现，故频率设置为1
            if(map.get(i)==null)
                map.put(i,1);
            else{
                //若当前元素非第一次出现，则出现频率在之前的基础上加1
                map.put(i,map.get(i)+1);
            }
        }
        //对元素按照频率进行降序排序
        List<Map.Entry<Integer,Integer>> list=new ArrayList<>(map.entrySet());
        Collections.sort(list, new Comparator<Map.Entry<Integer, Integer>>() {
            @Override
            public int compare(Map.Entry<Integer, Integer> o1, Map.Entry<Integer, Integer> o2) {
                return o2.getValue()-o1.getValue();
            }
        });
        //取出前K个元素
        int count=0;
        List<Integer> list1=new ArrayList<>();
        for(Map.Entry<Integer,Integer> entry:list){
            list1.add(entry.getKey());
            ++count;
            if(count>=k)
                break;
        }
        return list1;
    }
    public static void main(String[] args) {
        Integer[] nums=new Integer[]{1,1,1,2,2,3};
        List<Integer> list=new Solution2().topKFrequent(nums,2);
        System.out.println(list);
    }
}
//[1,2]
```

#### 最大堆---自定义优先级队列

因优先级队列是基于堆实现的，故需首先创建最大堆实现基本功能(增加元素、取出元素等)，之后自定义优先级队列依赖堆实现基本功能(入队、出队、堆顶元素、是否为空、堆中元素个数)，最后根据该题三步解题思路做出该题。

```java
//Heap.java
package heap;

import java.util.Arrays;
import java.util.Comparator;

//基于数组实现的二叉堆
public class Heap<E> {
    //实现任意类的大小比较
    private Comparator<E> comparator;
    private int size;
    private E[] elementData;
    //默认初始容量
    private static final int DEFAULT_CAPACITY=10;

    public Heap() {
        this(null,DEFAULT_CAPACITY);
    }

    public Heap(int initialCapacity) {
        this(null,initialCapacity);
    }

    public Heap(Comparator<E> comparator, int initialCapacity) {
        this.elementData= (E[]) new Object[initialCapacity];
        this.comparator=comparator;
    }
    public Heap(E[] arr){
        elementData= (E[]) new Object[arr.length];
        for(int i=0;i<arr.length;i++){
            elementData[i]=arr[i];
        }
        size=elementData.length;
        //Heapify
        for(int i=(arr.length-1-1)/2;i>=0;i--){
            siftDown(i);
        }
    }
    public int getSize(){
        return size;
    }
    public boolean isEmpty(){
        return size==0;
    }

    /**
     * 比较两个元素的大小
     * @param o1
     * @param o2
     * @return
     */
    private int compare(E o1,E o2){
        if(comparator==null){
            //此时E必须为Compareable接口子类
            return ((Comparable<E>)o1).compareTo(o2);
        }
        return comparator.compare(o1,o2);
    }

    /**
     * 返回当前节点的左孩子下标
     * @param index
     * @return
     */
    private int leftChildIndex(int index){
        return 2*index+1;
    }

    /**
     * 返回当前节点的右孩子下标
     * @param index
     * @return
     */
    private int rightChildIndex(int index){
        return 2*index+2;
    }
    private int parentIndex(int index){
        if(index==0)
            throw new IllegalArgumentException("你没有爸爸");
        return (index-1)/2;
    }
    public void add(E e){
        if(size==elementData.length){
            grow();
        }
        //向数组末尾添加元素
        elementData[size++]=e;
        //siftUp
        siftUp(size-1);//size++下标是下一位
    }
    private void siftUp(int index){
        while (index>0 && compare(elementData[index],elementData[parentIndex(index)])>0){
            //交换当前节点与父节点的值
            swap(index,parentIndex(index));
            //下一次比较时，当前index变为父节点下标，再次与当前节点的父节点比较大小
            index=parentIndex(index);
        }
    }
    private void swap(int indexA,int indexB){
        E temp=elementData[indexA];
        elementData[indexA]=elementData[indexB];
        elementData[indexB]=temp;
    }
    private void grow(){
        int oldCap=elementData.length;
        int newCap=oldCap+(oldCap<64?oldCap:oldCap>>1);
        if(newCap>Integer.MAX_VALUE-8)
            throw new ArrayIndexOutOfBoundsException("数组太大");
        elementData=Arrays.copyOf(elementData,newCap);
    }

    @Override
    public String toString() {
        StringBuilder res=new StringBuilder();
        for(E e:elementData){
            if(e!=null){
                res.append(e+" ");
            }
        }
        return res.toString();
    }
    public E findMax(){
        if(isEmpty())
            throw new IllegalArgumentException("heap is empty");
        return elementData[0];
    }
    public E extractMax(){
        E result=findMax();
        //交换堆顶元素与数组最后一个元素
        swap(0,size-1);
        elementData[--size]=null;
        //siftDowm
        siftDown(0);
        return result;
    }

    /**
     * 将二叉树当前节点下沉到正确位置
     * @param index
     */
    private void siftDown(int index){
        while (leftChildIndex(index)<size){
            int j=leftChildIndex(index);
            if(j+1<size){
                //此时该节点同时拥有左右孩子
                if(compare(elementData[j],elementData[j+1])<0)
                    j++;
            }
            //此时j为左右子树中最大值的下标
            //比较当前节点与左右子树中最大值的大小，若当前节点大于最大值，则停止交换
            if(compare(elementData[index],elementData[j])>0)
                break;
            swap(index,j);//将当前节点与左右子树中的最大值交换
            index=j;//进行下一次交换准备
        }
    }

    /**
     * 返回替换前的堆顶元素
     * @param newValue
     * @return
     */
    public E repalce(E newValue){
        E result=findMax();
        elementData[0]=newValue;
        siftDown(0);
        return result;
    }
}

//Queue.java
package heap;

public interface Queue<E> {
    void enQueue(E e);
    E deQueue();
    E peek();
    int size();
    boolean isEmpty();
}

//PriorityQueue.java
package heap;
//基于大顶堆的优先级队列
public class PriorityQueue<E> implements Queue<E> {
    Heap<E> heap=new Heap<>();
    @Override
    public void enQueue(E e) {
        heap.add(e);
    }

    @Override
    public E deQueue() {
        return heap.extractMax();
    }

    @Override
    public E peek() {
        return heap.findMax();
    }

    @Override
    public int size() {
        return heap.getSize();
    }

    @Override
    public boolean isEmpty() {
        return heap.isEmpty();
    }
}

//Solution.java
package heap;

import java.util.*;

/**
 * 前K个高频元素
 *
 * 1 1-3
 * 2 2-2
 * 3 3-1
 * TreeMap:添加了所有元素及其对应的频率
 * 优先级队列中存储频率最高的k个元素
 * 刚开始queue.size==0
 * queue.size最多等于k，当queue.size<k时一直queue.add()即可，当queue.size==k时，
 * 继续遍历Treemap中剩余数字频率，只要大于queue中已存在的最小值即可交换
 *
 * 每次替换优先队列中出现频率最小的元素
 *
 * 因此时我们实现的是最大堆，与正常逻辑相反，故我们将前k个高频元素按照从小到大的顺序输出，最初输出时即为正向
 */
public class Solution {
    private class Freq implements Comparable<Freq>{
        private int key;
        private int freq;
        public Freq(int key, int freq) {
            this.key = key;
            this.freq = freq;
        }
        @Override
        public int compareTo(Freq o) {
            if(this.freq<o.freq)
                return 1;
            if(this.freq>o.freq)
                return -1;
            return 0;
        }
    }
    public List<Integer> topKFrequent(int[] nums, int k){
        Map<Integer,Freq> map=new HashMap<>();
        //使用Map集合存储每个元素对应的频率
        for(int i:nums){
            if(map.get(i)==null){
                map.put(i,new Freq(i,1));
            }else{
                Freq freq=map.get(i);
                freq.freq+=1;
                map.put(i,freq);
            }
        }
        //使用优先级队列存储出现频率较大的前K个元素
        PriorityQueue<Freq> priorityQueue=new PriorityQueue<>();
        //开始遍历Map
        for(int key:map.keySet()){
            //当优先级队列元素未到K
            if(priorityQueue.size()<k){
                priorityQueue.enQueue(new Freq(key,map.get(key).freq));
            }
            //优先级队列元素达到K之后，每次替换队列中频率最小的元素
            if(map.get(key).freq>priorityQueue.peek().freq){
                //将优先级队列中最小的频率替换
                priorityQueue.deQueue();
                priorityQueue.enQueue(new Freq(key,map.get(key).freq));
            }
        }
        LinkedList<Integer> list=new LinkedList<>();
        while (!priorityQueue.isEmpty()){
            list.addFirst(priorityQueue.deQueue().key);
        }
        return list;
    }

    public static void main(String[] args) {
        int[] nums=new int[]{1,1,1,2,2,3};
        List<Integer> list=new Solution().topKFrequent(nums,2);
        System.out.println(list);
    }
}
//[1,2]
```

#### 最小堆---借助JDK自带的优先级队列实现(默认的优先级队列底层为最小堆)

```java
package heapTask;

import java.util.*;
import java.util.PriorityQueue;

/**
 * 最小堆实现
 * 时间复杂度：O(nlogn)，其中n表示数组长度。
 * 首先，遍历一遍数组统计元素的频率，这一系列操作的时间复杂度是O(n)的；
 * 接着，遍历用于存储元素频率的map，如果元素的频率大于最小堆中顶部的元素，则将顶部的元素删除并将该元素入堆中，这一系列
 * 操作的时间复杂度是O(nlogn)的；
 * 最后，弹出框堆中的元素所需的时间复杂度是O(nlogn)的。
 * 因此，总的时间复杂度是O(nlogn)
 * 空间复杂度：O(n)
 * 最坏情况下(每个元素都不同)，map需要存储n个键值对，优先队列需要存储k个元素，因此空间复杂度是O(n)
 */
public class Solutions {
    public List<Integer> topKFrequent(Integer[] nums,int k){
        Map<Integer,Integer> map=new HashMap<>();
        for(int i:nums){
            if(map.get(i)==null)
                map.put(i,1);
            else{
                Integer x=map.get(i);
                map.put(i,k+1);
            }
        }
        PriorityQueue<Integer> priorityQueue=new PriorityQueue<>(new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return map.get(o1)-map.get(o2);
            }
        });
        for(Integer key:map.keySet()){
            if(priorityQueue.size()<k)
                priorityQueue.add(key);
            else if(map.get(key)>map.get(priorityQueue.peek())){
                priorityQueue.remove();
                priorityQueue.add(key);
            }
        }
        List<Integer> list=new ArrayList<>();
        while (!priorityQueue.isEmpty()){
            list.add(priorityQueue.remove());
        }
        return list;
    }

    public static void main(String[] args) {
        Integer[] nums=new Integer[]{1,1,1,2,2,3};
        List<Integer> list=new Solutions().topKFrequent(nums,2);
        System.out.println(list);
    }
}
//[1,2]
```

#### 桶排序

```java
package heapTask;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 桶排序
 * 时间复杂度：O(n),其中n表示数组长度
 * 空间复杂度：O(n)
 */
public class Solution3 {
    public List<Integer> topKFrequent(Integer[] nums, int k){
        Map<Integer,Integer> map=new HashMap<>();
        for(int i:nums){
            if(map.get(i)==null)
                map.put(i,1);
            else
                map.put(i,map.get(i)+1);
        }
        //桶排序
        List<Integer>[] list=new List[nums.length+1];
        for(Integer key:map.keySet()){
            int freq=map.get(key);
            if(list[freq]==null){
                list[freq]=new ArrayList<>();
            }
            list[freq].add(key);
        }
        //逆序(频率由高到低)取出元素
        List<Integer> ret=new ArrayList<>();
        for(int i=nums.length;i>=0 && ret.size()<k;--i){
            if(list[i]!=null)
                ret.addAll(list[i]);
        }
        return ret;
    }
    public static void main(String[] args) {
        Integer[] nums=new Integer[]{1,1,1,2,2,3};
        List<Integer> list=new Solution2().topKFrequent(nums,2);
        System.out.println(list);
    }
}
//[1,2]
```

