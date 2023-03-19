# 排序

1. 数组排序

``` Java
    Integer[] nums;
    Arrays.sort(nums);

    Arrays.sort(res, new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
                return o1 - o2;
        }
    });
    
```

注意，使用以上两种方法时，数组必须是包装类型，否则会编译不通过
2. 二维数组排序

``` java
Arrays.sort(arr, new Comparator<int[]>() { 
       // 匿名内部类
	@Override
	public int compare(int[] e1, int[] e2) { 
   
		// 如果第一列元素相等，则比较第二列元素
		if (e1[0]==e2[0]) return e1[1]-e2[1];   // e1[1]-e2[1]表示对于第二列元素进行升序排序
		return e1[0]-e2[0];                     // e1[0]-e2[0]表示对于第一列元素进行升序排序
	}
});
```
