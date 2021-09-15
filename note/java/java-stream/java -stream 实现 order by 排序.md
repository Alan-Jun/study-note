有这样一个排序sql,  现在要在java代码中实现

```
ORDER BY `status` desc, update_time desc 
```

我们在使用 sorted 方法排序的时候需要注意将order by 字段倒序使用，因为 java stream  sorted 排序实现方式方式需要这样去调用

```
list.stream().sorted(Comparator.comparing(LoginDeviceDO::getUpdateTime).reversed())
    .sorted(Comparator.comparing(LoginDeviceDO::getStatus).reversed()).collect(Collectors.toList());
```

