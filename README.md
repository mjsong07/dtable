# element table 动态表格与行编辑
基于elementPlus的el-table + vite + vue3 + jsx 实现的动态表格和行编辑demo

# 安装与运行
```sh
npm i
npm run dev
```

# 使用方式

```js
//定义列格式
let columns = 
[ 
  { label: '城市', property: 'city', render:'component',  type: 'select',  options: cityOptions , validate:true },  
  { label: '介绍', property: 'info', render:'component',width:120,  type: 'input'},   
]
//定义表格数据
let tableData = [
{"id":2,"city":1,"area":1,"username":3,"account":4,"age":5,"createtime":"2024-01-01",birthday:"202405",score:5,info:'人品好',readonlyKeyList:[]},
{"id":3,"city":2,"area":3,"username":3,"account":4,"age":5,"createtime":"2024-01-02",birthday:"202406",score:-2,info:'孤僻',readonlyKeyList:[],ignoreList:['birthday']}, 
]

//组件使用
  <DTable ref="tableRef" rowKey="id"  :tableData="tableData" :columns="columns"></DTable>   

//表格某一行提交
const handleRowSubmit = (row:any) => { 
  let {isValidate,errorMsg,errorColList} = tableRef.value?.validate(row)
  if(!isValidate){//验证不通过
    row.errorList = errorColList //错误列设置，用于标识红框提示
    const warnMessage = errorMsg.join('<br/>') 
    // 输出错误提示 ....warnMessage
    return
  } 
}

```

## DTable介绍

### columns 数据结构介绍
```js
property //对应表格的列属性
render 'component' | 'slot'; // 可选的固定组件，也可以选择自定义渲染方式， 不传则按员el-table的方式渲染
validate //是否需要校验
```

### tableData数据内容介绍
```js
uiEdit: true | false; //是否可编辑
readonlyKeyList: [] //某一行限制部分列只读
ignoreList: []   // 某一行取消校验限制
errorList:  []   // 某一行数据异常标识
```

# 实现功能：
1. 基于配置生成表格
2. 行内支持编辑和查看 两种状态显示
3. 支持添加，删除，编辑行
4. 支持必填校验和批量提示，部分校验忽略
5. 支持自定义模板
6. 支持的显示组件有 input 输入框 ， select（当超过1000条切换到虚拟滚动select）下拉控件 ， date-pick 日期控件
7. 输入框支持数字，正负，小数位显示
8. 支持回调事件，可实现不同组件间的联通，如省市区联动逻辑
9. 日期控件支持格式化显示
