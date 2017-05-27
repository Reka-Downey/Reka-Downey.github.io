---
layout: post
title: 通用分页封装模型
category: 2017年5月
tags: Paging
keywords: Paging
description: 通用分页封装模型
---

## 说明

　　分页是开发中的一种常用技术，`Spring Data`提出的分页模型比较复杂，有`Slice`、`Chunk`、`Page`等概念，见下图。

![SpringDataPaging](https://ooo.0o0.ooo/2017/05/27/5928e918cb7a0.jpg)

　　由于存在不需要`Spring Data`参与的项目，因此有了将该分页模型提炼出来的想法。

　　本文的分页模型借鉴了`Spring Data`的实现，同时加入了几个自认为比较常用的字段以增强分页效果。本文主要介绍提炼后的分页效果，不提及实现过程。

---

## 分页模型说明

　　该分页模型的主要字段：

```java
public class Page<T> implements Serializable {

    /**
     * 当前查询实际拥有的数据量，会根据偏移量、页面数据量以及当前页内容数量进行微调
     */
    private final long totalElements;

    /**
     * 当前查询总共拥有的页数
     */
    private final int totalPages;

    /**
     * 当前请求的页码偏移量
     */
    private final int reqPageOffset;

    /**
     * 当前请求的页面数据量，即要求每页显示多少条数据
     */
    private final int reqPageSize;

    /**
     * 当前页的页码偏移量，0 代表 第一页，1 代表第二页，正常情况下该值必然与 {@link #reqPageOffset} 一致
     */
    private final int curPageOffset;

    /**
     * 当前页的页面数据量，在尾页的时候，该值可能与 {@link #reqPageSize} 不等
     */
    private final int curPageSize;

    /**
     * 当前请求的页码导航数量，即可以导航到前/后多少条
     */
    private final int reqPagingNavigationNum;

    /**
     * 当前页是否有上一页
     */
    private final boolean hasPreviousPage;

    /**
     * 当前页是否有下一页
     */
    private final boolean hasNextPage;

    /**
     * 当前页是否是首页
     */
    private final boolean isFirstPage;

    /**
     * 当前页是否是尾页
     */
    private final boolean isLastPage;

    /**
     * 在导航栏中，符合 {@link #reqPagingNavigationNum} 约束下，当前页的前面能够存在多少个导航页码。
     * <pre>
     * 　　例如：当前{@link #reqPagingNavigationNum} 为 10，{@link #totalPages} 为 20，那么：
     * 假设当前为第 5 页，导航栏中将只有 1、2、3、4 共 4 条前置分页导航页码；
     * 而若当前为第 15 页，导航栏中将有 5、6、7、8、9、10、11、12、13、14 共 10 条前置分页导航页码；
     * </pre>
     */
    private final int[] previousNavigation;

    /**
     * 在导航栏中，符合 {@link #reqPagingNavigationNum} 约束下，当前页的后面能够存在多少个导航页码。
     * <pre>
     * 　　例如：当前{@link #reqPagingNavigationNum} 为 10，{@link #totalPages} 为 20，那么：
     * 假设当前为第 5 页，导航栏中将有 6、7、8、9、10、11、12、13、14、15 共 10 条后置分页导航页码；
     * 而若当前为第 15 页，导航栏中将只有 16、17、18、19、20 共 5 条后置分页导航页码；
     * </pre>
     */
    private final int[] nextNavigation;

    /**
     * 当前页的页面数据
     */
    private final List<T> content;

}
```

　　该分页通过内部创建者模式来构造，具体的创建者行为包括：

```java
public static class Builder<T> {

    private long totalElements;
    private int totalPages;
    private PageRequest pageRequest;
    private int reqPageOffset;
    private int reqPageSize;
    private int reqPagingNavigationNum = PageRequest.DEFAULT_NAVIGATION_NUM;
    private int curPageOffset;
    private int curPageSize;
    private int[] previousNavigation;
    private int[] nextNavigation;
    private boolean hasPreviousPage;
    private boolean hasNextPage;
    private boolean isFirstPage;
    private boolean isLastPage;
    private List<T> content;

    public Builder(List<T> content) {
        this.content = content;
    }

    public Builder<T> pageOffset(final int pageOffset) {
        this.reqPageOffset = pageOffset;
        return this;
    }

    public Builder<T> pageSize(final int pageSize) {
        this.reqPageSize = pageSize;
        return this;
    }

    public Builder<T> pagingNavigationNum(final int pagingNavigationNum) {
        this.reqPagingNavigationNum = pagingNavigationNum;
        return this;
    }

    public Builder<T> pageRequest(final PageRequest pageRequest) {
        this.pageRequest = Objects.requireNonNull(pageRequest);
        return this;
    }

    public Builder<T> totalElements(final long totalElements) {
        this.totalElements = totalElements;
        return this;
    }

    public Builder<T> content(List<T> content) {
        this.content = Objects.requireNonNull(content);
        return this;
    }
  
    public Page<T> build() {
        return create();
    }
}
```

---

## 分页测试

　　测试样例：

```java
    @Test
    public void testPage() throws Exception {
        List<String> content = new ArrayList<>(Arrays.asList("Reka", "雷卡", "Rachel", "瑞秋"));
        Page<String> page = Page.custom(content)
                                .pageOffset(1)
                                .pageSize(5)
                                // 根据 pageOffset、pageSize、content.size()，totalElements 被调整成 9
                                .totalElements(8)
                                .pagingNavigationNum(3)
                                .build();
        Gsonor.PRETTY.toJson(page, System.out);

        List<User> userContent = new ArrayList<>();
        userContent.add(new User("Reka", 23));
        userContent.add(new User("雷卡", 23));
        userContent.add(new User("Rachel", 22));
        userContent.add(new User("瑞秋", 22));
        Page<User> userPage = new Page.Builder<>(userContent)
                // pageRequest 的优先级高于 pagingNavigationNum、pageOffset、pageSize，后面这几项配置都不生效
                .pageRequest(new PageRequest(3, 4, 4))
                .pagingNavigationNum(5)
                .totalElements(30)
                .pageOffset(3)
                .pageSize(7)
                .build();
        Gsonor.PRETTY.toJson(userPage, System.out);
    }
```

　　测试结果：

```json
// 字符串分页
{
  "totalElements": 9,
  "totalPages": 2,
  "reqPageOffset": 1,
  "reqPageSize": 5,
  "curPageOffset": 1,
  "curPageSize": 4,
  "reqPagingNavigationNum": 3,
  "hasPreviousPage": true,
  "hasNextPage": false,
  "isFirstPage": false,
  "isLastPage": true,
  "previousNavigation": [
    1
  ],
  "nextNavigation": [],
  "content": [
    "Reka",
    "雷卡",
    "Rachel",
    "瑞秋"
  ]
}

// 对象分页
{
  "totalElements": 30,
  "totalPages": 8,
  "reqPageOffset": 3,
  "reqPageSize": 4,
  "curPageOffset": 3,
  "curPageSize": 4,
  "reqPagingNavigationNum": 4,
  "hasPreviousPage": true,
  "hasNextPage": true,
  "isFirstPage": false,
  "isLastPage": false,
  "previousNavigation": [
    1,
    2,
    3
  ],
  "nextNavigation": [
    5,
    6,
    7,
    8
  ],
  "content": [
    {
      "name": "Reka",
      "age": 23
    },
    {
      "name": "雷卡",
      "age": 23
    },
    {
      "name": "Rachel",
      "age": 22
    },
    {
      "name": "瑞秋",
      "age": 22
    }
  ]
}
```

---

　　该分页模型比较简单，由`Page`和`PageRequest`两个类组成，没有第三方依赖。要求`JDK`版本为`1.5`及以上。

源码地址：[GitHub](https://github.com/Reka-Downey/paging) 



|                 |
| --------------- |
| 编写日期：2017-05-27 |
| 发布日期：2017-05-27 |



