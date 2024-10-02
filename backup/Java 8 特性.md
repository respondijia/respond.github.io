## stream / parallelstream流式处理
parallelStream() 方法可以将集合数据分成多个小块，分配到多个线程并行处理，从而提高程序的执行效率。
### parallelstream流式处理的坑
具体详情：https://blog.csdn.net/lsx2017/article/details/105749984#:~:text=java8%E4%B8%AD%E7%9A%84%E6%96%B0
## Optional 可选类
比如判空处理：
```java 
// 内存中判断是否含有包含标签
        return userList.stream().filter(user -> {
            String userTagsJson = user.getTags();
            List<String> tempUserTags = JSONUtil.toList(userTagsJson, String.class);
            // 判空
            List<String> userTags = Optional.ofNullable(tempUserTags).orElse(new ArrayList<>());
            for (String s : tagNameList) {
                if(!userTags.contains(s)){
                    return false;
                }
            }
            return true;
        }).map(this::getSafetyUser).collect(Collectors.toList());
```
