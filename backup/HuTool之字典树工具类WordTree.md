先定义一个黑白名单，比如哪些操作是禁止的，可以就是一个列表：

```java
private static final List<String> blackList = Arrays.asList("Files", "exec");
```

还可以使用字典树代替列表存储单词，用 更少的空间 存储更多的敏感词汇，并且实现 更高效 的敏感词查找。

此处使用 HuTool 工具库的字典树工具类：WordTree。

1）先初始化字典树，插入禁用词：

```java
private static final WordTree WORD_TREE;
static {
    // 初始化字典树
    WORD_TREE = new WordTree();
    WORD_TREE.addWords(blackList);
}
```


2）校验用户代码是否包含禁用词：

```java
//  校验代码中是否包含黑名单中的禁用词
FoundWord foundWord = WORD_TREE.matchWord(code);
if (foundWord != null) {
    System.out.println("包含禁止词：" + foundWord.getFoundWord());
    return null;
}
```