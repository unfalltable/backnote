## 三级分类

### 代码

```java
private List<Tree<Long>> getCategoryTree(List<CategoryEntity> categorys, Long parentId) {
    List<TreeNode<Long>> collect = categorys.stream().filter(category -> category.getCatId().intValue() != category.getParentCid())
        .sorted(Comparator.comparingInt(CategoryEntity::getSort)).map(category -> {
        TreeNode<Long> treeNode = new TreeNode<>();
        treeNode.setId(category.getCatId());
        treeNode.setParentId(category.getParentCid());
        treeNode.setName(category.getName());
        treeNode.setWeight(category.getSort());
        //扩展属性
        Map<String, Object> extra = new HashMap<>(1);
        //extra.put("", );
        treeNode.setExtra(extra);
        return treeNode;
    }).collect(Collectors.toList());
    return TreeUtil.build(collect, parentId);
}
```

