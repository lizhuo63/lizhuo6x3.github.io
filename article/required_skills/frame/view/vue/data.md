# 获取表格中某一整列的数据

```
通过map方法，流式取出[v-model]tableData 中prop【tableName】的列数据
let ts = app.tableData.map(element => element.tableName);
```