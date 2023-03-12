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
