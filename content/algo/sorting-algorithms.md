+++
title = "排序算法 <一>"
categories = ["算法"]
tags = ["排序算法","基础"]
isCJKLanguage = true
date = "2017-02-13T15:21:01+08:00"

+++

就让我再啃一下一直以来最头疼的问题吧。

## 冒泡排序(bubble sorting)

刚才突然想通了，之前竟然不知道为什么大家都说冒泡是最简单的。。。难道是我智商不够了么

假设有这样一组数字$a = [2, 3, 8, 1, 5, 7]$。要对它进行排序，按照冒泡排序的做法最终是要这样做：

1. 设数组的索引是$i$，如果$a[i] > a[i+1]$，那么就让二者交换位置

2. 紧接着让$a[i+1]$和$a[i+2]$做比较，要知道这时$a[i+1]$已经是$a[i]$和$a[i+1]$中较大的了，按照前面的做法把二者中较大的放在后面

3. 这样做一轮最大的数字就会在新数组最后了

4. 但前面5位的位置依然是乱序的，要按前面3步的规则重新排一遍。因为最后一位已经是最大无疑，就不需要参与排序了

5. 第一轮排序得到一个最大的，第二轮得到第二大，当第二小的确定以后，第一小的也随之确定，所以一共需要做$n-1$轮排序

   ​

   原理清楚了，用代码把它描述出来就是这样了（为了突出一个逼格，一定要用C啊）：

   ```c
   #include <stdio.h>

   void swap(int a[], int i, int j)
   {
       int tmp = a[i];
       a[i] = a[j];
       a[j] = tmp;
   }

   void bubble_sort(int a[], int len)
   {
       for (int j = 0; j < len - 1; j++) {
           for (int i = 0; i < len - 1 - j; i++) {
               if (a[i] > a[i+1]) {
                   swap(a, i, i+1);
               }
           }
       }
   }

   int main()
   {
       int a[] = {2, 3, 8, 1, 5, 7};
       int n = sizeof(a) / sizeof(int);
       int i;

       puts("原始数组: ");
       for (i = 0; i < n; i++) {
           printf("%d\t", a[i]);
       }
       puts("");
       printf("数组长度: %d\t", n);

       bubble_sort(a, n);

       puts("排序后数组: ");
       for (i = 0; i < n; i++) {
           printf("%d\t", a[i]);
       }
       puts("");

       return 0;
   }
   ```

   ​

   为了让自己不忘记最核心的代码，这里详细的描述一下：

   1. 最外层循环是说要进行`n-1`轮排序才能完成
   2. 内层循环是说外层循环每进行一轮，需要进行比较的数字就会少一组，也就是前面提到的，最大值已经确定了，后面的排序它就不需要参与了
   3. 然后就是最简单的交换顺序了

   这样理解下来，冒泡排序果然是最简单的了。

   再呈上PHP版本：

   ```php
   <?php

   function swap(array $a, $i, $j)
   {
       $tmp = $a[$i];
       $a[$i] = $a[$j];
       $a[$j] = $tmp;

       return $a;
   }

   function bubbleSort(array $a, $n)
   {
       for ($i = 0; $i < $n; $i++) {
           for ($j = 0; $j < $n - 1 - $i; $j++) {
               if ($a[$j] > $a[$j+1]) {
                   $a = swap($a, $j, $j+1);
               }
           }
       }

       return $a;
   }

   $arr = [2, 3, 8, 1, 5, 7];
   $b = bubbleSort($arr, 6);
   var_export($b);

   ```




最近新学了golang，顺道也写一遍：

```go
   package main

   import "fmt"

   func swap(a []int, i int, j int) {
   	tmp := a[i]
   	a[i] = a[j]
   	a[j] = tmp
   }

   func bubbleSort(a []int, n int) {
   	for i := 0; i < n-1; i++ {
   		for j := 0; j < n-1-i; j++ {
   			if a[j] > a[j+1] {
   				swap(a, j, j+1)
   			}
   		}
   	}
   }

   func main() {
   	a := []int{2, 3, 8, 1, 5, 7}
   	bubbleSort(a, 6)
   	fmt.Println(a)
   }
```


## 鸡尾酒排序(cocktail sorting)

鸡尾酒排序是冒泡排序的一种改进。前面说了，冒泡排序每轮排序仅仅选出其中最大的，而鸡尾酒排序则是在选出最大的以后，把最小的也选出来放在最前面。冒泡排序的时间复杂度是$(O(n^2)$，这样看起来鸡尾酒排序的时间复杂度应该是$O(\frac{n^2}{2})$，但没有那么记复杂度的，它和冒泡排序的时间复杂度是同一个数量级，但当需要排序的数字比较少时，通常鸡尾酒排序能获得更好的性能。

```c
#include <stdio.h>

void swap(int a[], int i, int j)
{
    int tmp = a[i];
    a[i] = a[j];
    a[j] = tmp;
}

void cocktailSort(int a[], int n)
{
    int left = 0;
    int right = n - 1;
    while (left < right) {
        for (int i = left; i < right; i++) {
            if (a[i] > a[i+1]) {
                swap(a, i, i+1);
            }
        }
        right--;

        for (int i = right; i > left; i--) {
            if (a[i-1] > a[i]) {
                swap(a, i-1, i);
            }
        }
        left++;
    }
}

int main()
{
    int a[] = {2, 3, 8, 1, 5, 7};
    int n = sizeof(a) / sizeof(int);
    int i;

    puts("原始数组: ");
    for (i = 0; i < n; i++) {
        printf("%d\t", a[i]);
    }
    puts("");
    printf("数组长度: %d\n", n);

    cocktailSort(a, n);

    puts("排序后数组: ");
    for (i = 0; i < n; i++) {
        printf("%d\t", a[i]);
    }
    puts("");

    return 0;
}
```

老规矩，PHP版本

```php
<?php

function swap(array $a, $i, $j)
{
    $tmp = $a[$i];
    $a[$i] = $a[$j];
    $a[$j] = $tmp;

    return $a;
}

function cocktailSort(array $a, $len)
{
    $left = 0;
    $right = $len - 1;

    while ($left < $right) {
        for ($i = $left; $i < $right; $i++) {
            if ($a[$i] > $a[$i+1]) {
                $a = swap($a, $i, $i+1);
            }
        }
        $right--;

        for ($i = $right; $i > $left; $i--) {
            if ($a[$i-1] > $a[$i]) {
                $a = swap($a, $i-1, $i);
            }
        }
        $left++;
    }

    return $a;
}


$arr = [2, 3, 8, 1, 5, 7];

$b = cocktailSort($arr, 6);
var_export($b);
```



golang版本：



```go
package main

import "fmt"

func swap(a []int, i int, j int) {
	tmp := a[i]
	a[i] = a[j]
	a[j] = tmp
}

func cocktailSort(a []int, len int) {
	left := 0
	right := len - 1

	for left < right {
		for i := left; i < right; i++ {
			if a[i] > a[i+1] {
				swap(a, i, i+1)
			}
		}
		right--

		for i := right; i > left; i-- {
			if a[i-1] > a[i] {
				swap(a, i-1, i)
			}
		}
		left++
	}
}

func main() {
	arr := []int{2, 3, 8, 1, 5, 7}
	cocktailSort(arr, 6)
	fmt.Println(arr)
}
```



## 选择排序（selection sorting)

这个现在看起来好像也很直观了（下面我按大家通常说的『趟』来替代前面说的『轮』）。

1. 每趟找出一个最小的放在最前面
2. 如果有$n$个数，就要排$n-1$趟
3. 对于每趟排序而言，设置一个flag $min$，用它标识该趟排序最小数的下标，第$i$趟排序开始时，前第$i-1$个数已经确定了，所以把$min = i$，后面把第$min$位数和它后面的所有数字比较，如果$a[min] > a[min+1]$就把$min$设置为$min+1$
4. 这一步最为关键，第3步找到了这趟排序的最小数，这步就要把放在第$i$位了，那原来第$i$位的数就当然和它交换位置了

不多说，直接上代码：

```c
#include <stdio.h>

void swap(int a[], int i, int j)
{
    int tmp = a[i];
    a[i] = a[j];
    a[j] = tmp;
}

void selection_sort(int a[], int n)
{
    int min, i, j;

    for (i = 0; i < n - 1; i++) { // 0 -> (n-2) 一共(n-1)趟
        min = i;
        for (j = min + 1; j < n; j++) {
            if (a[min] > a[j]) {
                min = j;
            }
        }
        if (min != i) {
            swap(a, min, i);
        }
    }
}

int main()
{
    int a[] = {2, 3, 8, 1, 5, 7};
    int n = sizeof(a) / sizeof(int);
    int i;

    puts("原始数组: ");
    for (i = 0; i < n; i++) {
        printf("%d\t", a[i]);
    }
    puts("");
    printf("数组长度: %d\t", n);

    selection_sort(a, n);

    puts("排序后数组: ");
    for (i = 0; i < n; i++) {
        printf("%d\t", a[i]);
    }
    puts("");

    return 0;
}
```

PHP和golang版本的这里就不贴了，写完这篇文章单独搞一个github的repo来存放。

## 插入排序(insertion sorting)

这个就像玩扑克抓牌一样，以下是步骤：

1. 抓出一张牌，放在左手
2. 再抓出一张牌，从老牌的第一张开始扫描，如果新牌小于老牌，老牌往后移动一位，新牌替换老牌的位置
3. 在上面的扫描过程中，如果新牌大于老牌，则把新牌放在老牌下一位
4. 重复以上步骤

来看代码：

```c
#include <stdio.h>

void insertion_sort(int a[], int n)
{
    int i, j, get;

    for (i = 1; i < n; i++) {
        get = a[i];
        j = i - 1;
        while (j >= 0 && get < a[j]) {
            if (get < a[j]) {
                a[j+1] = a[j];
                j--;
            }
        }
        a[j+1] = get;
    }
}

int main()
{
    int a[] = {2, 3, 8, 1, 5, 7};
    int n = sizeof(a) / sizeof(int);
    int i;

    puts("原始数组: ");
    for (i = 0; i < n; i++) {
        printf("%d\t", a[i]);
    }
    puts("");
    printf("数组长度: %d\t", n);

    insertion_sort(a, n);

    puts("排序后数组: ");
    for (i = 0; i < n; i++) {
        printf("%d\t", a[i]);
    }
    puts("");

    return 0;
}
```

这个排序貌似比较难理解一些，我再详细的解释一下。

还是把这种排序类比成抓扑克牌。

1. 假设手里有一把牌($n$张），我们一般不会去移动第一张牌吧，所以从第二张牌开始，后面的每一张都需要被抓一下，所以外层循环是$n-1$。
2. 第一次以第一张作为基准，取第二张牌，所以内层循环从1开始而不是0。
3. $get$这个变量是为了暂存取出牌的值
4. 有一个很重要的前提**位于新取出的位置之前的所有牌都是已经排好序的**。所以我们拿新牌去和前面所有已经排序的牌做比较。假设$get = a[i]$，那么如果$a[i-1] > a[i]$，就需要让$a[i-1]$向后移动一位，也就是$a[i] = a[i-1]$，这就是需要$get$这个变量来暂存$a[i]$的原因了。如果前面若干位都大于$get$，那就一直移动到不大于它的为止。这时就会出现一个空缺，这个空缺就需要$get$来填充了

看看前面的代码是不是完美的诠释了这个过程？

## 二分插入排序

二分插入排序本质上和（直接）插入排序是一样的，只是在寻找$a[i]$时采用了二分查找法，二分查找发具体算法如下：

1. 给定的已排序的序列$b$，长度为$n$，$left$和$right$分别是它的起点和终点，即$left=0;right=n-1$，中间位置$mid=\frac{left+right}{2}$,给定的值$k$。
2. 首先拿序列$b$的中间值和$k$做比较，有以下两种情况:
    - $k > b[mid]$，则$k$的下标位于$b$的后半部分，$left = mid + 1$
    - $k < b[mid]$，则$k$的下标位于$b$的前半部分，$right = mid - 1$
3. 如果$left > right$，则查找失败

在我们插值的场景中当然是查找失败的，但这却是退出whilex循环的条件，也就是在while循环退出时,$left = right + 1$，而$b[left]$是第一个大于$k$的数。这时就要把$[left, i-1]$位置的所有元素都向后移动一位，然后把$a[left] = k$。好了，这样整个算法就很清晰了。上C代码：

```c
#include <stdio.h>

void binary_sort(int a[], int n)
{
    int i, j, right, left, get;

    for (i = 1; i < n; i++) {
        left = 0;
        right = i - 1;
        get = a[i];

        while (left <= right) {
            int mid = (left + right) / 2;
            if (a[mid] > get) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }

        for (j = i - 1; j >= left; j--) {
            a[j+1] = a[j];
        }

        a[left] = get;
    }
}

int main()
{
    int a[] = {2, 3, 8, 1, 5, 7};
    int n = sizeof(a) / sizeof(int);
    int i;

    puts("原始数组: ");
    for (i = 0; i < n; i++) {
        printf("%d\t", a[i]);
    }
    puts("");
    printf("数组长度: %d\t", n);

    binary_sort(a, n);

    puts("排序后数组: ");
    for (i = 0; i < n; i++) {
        printf("%d\t", a[i]);
    }
    puts("");

    return 0;
}
```

有了前面铺垫的描述，这代码写下来就是顺理成章啊。

## 希尔排序（shell sorting)

上面都是很直观的排序方法，下面介绍的几种就没那么直观了。首先是希尔排序。

希尔排序本质上还是插入排序的一种推广，它基于以下两条事实：

1. 插入排序在对**几乎已经排好序的数据**操作时，效率高，即可以达到**线性排序**的效率
2. 但插入排序一般来说是低效的，因为插入排序**每次只能将数据移动一位**

既然知道了插入排序的强项和弱项，就能更好的应用它以提升排序效率了。首先就是要将一个无序的数据构造成**更有序**的状态，也就是前面说的**几乎已经排好序的数据**。

首先把$n$个数分成$2$队，两队中各有$m$个元素。每个元素在本队中的位置和别的队中同样位置的元素构成一组。这样会产生类似$[A\_1, B\_1], [A\_2, B\_2],[A\_3, B\_3], [A\_4, B\_4]$这样的数据。然后把每组中的两个数据分别排序，按照**递归**的分组策略继续重新分组，把每组中的数据分别排序，最终间隔越来越小，变成1时也就完成了排序。

至于分组策略，一般的实现是从$m = n / 2$开始，持续$m = m / 2$最终$m=1$了。

然后每个组排序，也就有$m$个组。

每组中，按间隔为$m$做插入排序。

不断迭代到$m=1$就完成了排序。

下面是C代码：

```c
#include <stdio.h>

void shell_sort(int a[], int n)
{
    int m, i, j;

    for (m = n / 2; m > 0; m /= 2) {
        printf("m: %d\n", m);

        for (i = 0; i < m; i++) {
            for (j = i + m; j < n; j += m) {
                if (a[j] < a[j-m]) {
                    int get = a[j];
                    int k = j - m;
                    while (k >= 0 && a[k] > get) {
                        a[k+m] = a[k];
                        k -= m;
                    }
                    a[k+m] = get;
                }
            }
        }
    }
}

int main()
{
    int a[] = {2, 3, 8, 1, 5, 7};
    int n = sizeof(a) / sizeof(int);
    int i;

    puts("原始数组: ");
    for (i = 0; i < n; i++) {
        printf("%d\t", a[i]);
    }
    puts("");
    printf("数组长度: %d\t", n);

    shell_sort(a, n);

    puts("排序后数组: ");
    for (i = 0; i < n; i++) {
        printf("%d\t", a[i]);
    }
    puts("");

    return 0;
}
```

这个虽然说了很明白，但总还是不太好理解，如果能把直接插入排序理解成$m=1$的希尔排序的特殊情况就好理解一些了。