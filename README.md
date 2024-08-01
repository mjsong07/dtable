# element table 动态表格与行编辑
基于elementPlus的el-table + vite + vue3 + jsx 实现的动态表格和行编辑demo

# 实现功能：
1. 基于配置生成表格
2. 行内支持编辑和查看 两种状态显示
3. 支持添加，删除，编辑行
4. 支持必填校验和批量提示，部分校验忽略
5. 支持自定义模板
6. 支持的显示组件有 input 输入框 ， select（虚拟滚动select）下拉控件 ， date-pick 日期控件
7. 输入框支持数字，正负，小数位显示
8. 支持回调事件，可实现不同组件间的联通，如省市区联动逻辑
9. 日期控件支持格式化显示

# 使用方式

```js
//定义列格式
columns = 
[
  { label: 'ID', property: 'id',align:'center', type:'selection' , width:30},
  { label: '操作', property: 'slot', render:'slot',slotName:'operateSlot', width:100 ,fixed:'left'  }
]
//定义表格数据
let data = 
[
{"id":2,"city":1,"area":1,"username":3,"account":4,"age":5,"createtime":"2024-01-01",month:"202405","updatetime":"2024-04-01",readonlyKeyList:[]},
{"id":3,"city":2,"area":3,"username":3,"account":4,"age":5,"createtime":"2024-01-02",month:"202406","updatetime":"2024-04-02",readonlyKeyList:[]}, 
]
```