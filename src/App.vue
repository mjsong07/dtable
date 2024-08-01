<script setup lang="ts">
import DTable from './components/DTable.vue' 
import { reactive, ref, toRefs , nextTick } from 'vue';
import { ElMessage, ElMessageBox } from 'element-plus';
import { zhCn } from "element-plus/es/locale/index.mjs";
import {cloneDeep} from 'lodash'

//定义省份的数组
const cityOptions = [
  { label: '广州', value: 1},
  { label: '深圳', value: 2},
  { label: '北京', value: 3},
  { label: '上海', value: 4},
  { label: '杭州', value: 5},
  { label: '天津', value: 6},
  { label: '重庆', value: 7},
  { label: '南京', value: 8},
  { label: '香港', value: 9},
]

//定义市区的数组以来省份 补充多5
const areaOptions = [
  { label: '荔湾', value: 1, parantId:1},
  { label: '增城', value: 2, parantId:1},
  { label: '罗湖', value: 3, parantId:2},
  { label: '福田', value: 4, parantId:2},
  { label: 'xx1', value: 5, parantId:3}, 
  { label: 'xx2', value: 6, parantId:3}, 
  { label: 'xx3', value: 7, parantId:4}, 
  { label: 'xx4', value: 8, parantId:4}, 
  { label: 'xx5', value: 9, parantId:5}, 
]
//用户数组
const userOptions = [
  { label: '张三', value: 1},
  { label: '李四', value: 2},
  { label: '王五', value: 3},
  { label: '赵六', value: 4},
  { label: '钱七', value: 5},
  { label: '孙八', value: 6},
  { label: '周九',value: 7},
  { label: '周九1',value: 8},
  { label: '周九2',value: 9},
]


const usernameChange = (value:string,props:any) => {
  if( value === '') return  {isError:true,val:value}
  if(props.options.some((o:any)=> o.value === value)) {
    return false
  }
  let  regex = /^[\u4e00-\u9fa5]*$/; // 定义正则表达式
  if (value && !regex.test(value)) { 
    ElMessage.warning("只能输入中文")
    return  {isError:true,val:''}
  }
  return  {isError:true,val:value}
}

const getTableAreaOptions = (row:any,column:any) => { 
  return row.city ? areaOptions.filter( o => o.parantId === row.city) : areaOptions
} 

const tableRef = ref({})
const state = reactive({
  tableData:[],
  columns:[
  { label: 'ID', property: 'id',align:'center', type:'selection' , width:30},
  { label: '操作', property: 'slot', render:'slot',slotName:'operateSlot', width:100 ,fixed:'left'  }, 
  { label: '城市', property: 'city', render:'component',  type: 'select',  options: cityOptions , validate:true },  
  //市区与省份联动，通过
  { label: '区域', property: 'area', render:'component',  type: 'select',  options: getTableAreaOptions , validate:true },  
  //支持下拉动态创建（输入后回车），以及输入内容验证控制
  { label: '用户', property: 'username', render:'component',  type: 'select',width:120,   options: userOptions , attr: {allowCreate:true,onPropValChange:usernameChange}},
  //限制正数输入,保留两位小数
  { label: '钱包', property: 'account', render:'component',  type: 'input', attr: {type:'number',number:'float',precision:2} }, 
  //限制正数输入
  { label: '年龄', property: 'age', render:'component',  type: 'input', attr: {type:'number',number:'positive'} , validate:true},  
  //可以输入负数，限制最大最小值
  { label: '学分', property: 'score', render:'component',  type: 'input', attr: {type:'number',number:'positiveNegative',max:100,min:-100} },  
  { label: '生日年月', property: 'birthday', render:'component',  type: 'date' ,width:120, attr:{type:'month',format:'YYYYMM',valueFormat:'YYYYMM'} , validate:true}, 
  { label: '介绍', property: 'info', render:'component',width:120,  type: 'input'},  
  { label: '提交日期', property: 'createtime',width:180},
]})
const {columns,tableData} = toRefs(state)

// tableData数组设置 以columns的property数据为属性的数据 
let data = [
{"id":2,"city":1,"area":1,"username":3,"account":4,"age":5,"createtime":"2024-01-01",birthday:"202405",score:5,info:'人品好'},
{"id":3,"city":2,"area":3,"username":3,"account":4,"age":5,"createtime":"2024-01-02",birthday:"202406",score:-2,info:'孤僻',ignoreList:['birthday']}, 
{"id":4,"city":3,"area":5,"username":3,"account":4,"age":5,"createtime":"2024-01-03",birthday:"202407",score:10,info:'吝啬',readonlyKeyList:['city', 'area',]},//支持设定某一行部分字段临时不能编辑
{"id":5,"city":4,"area":2,"username":3,"account":4,"age":5,"createtime":"2024-01-04",birthday:"202408",score:20,info:'大方'},
{"id":6,"city":5,"area":2,"username":3,"account":4,"age":5,"createtime":"2024-01-05",birthday:"202409",score:-1,info:'穷'},
{"id":7,"city":6,"area":2,"username":3,"account":4,"age":5,"createtime":"2024-01-06",birthday:"202410",score:-2,info:'有钱'},
{"id":8,"city":7,"area":2,"username":3,"account":4,"age":5,"createtime":"2024-01-07",birthday:"202405",score:-3,info:'凤凰男'},
{"id":9,"city":8,"area":2,"username":3,"account":4,"age":5,"createtime":"2024-01-08",birthday:"202405",score:-4,info:'单身'},
{"id":10,"city":9,"area":2,"username":3,"account":4,"age":5,"createtime":"2024-01-09",birthday:"202405",score:20,info:'双身'}];

  tableData.value = data  
  let copyTableData =  cloneDeep(data)

  const batchDelete = () => { 
    if(!multipleSelection.value) { 
      ElMessage.warning("请先勾选数据")
      return
    }
    ElMessageBox.confirm('是否确认删除数据？')
    .then(function () { 
     const indexesToRemove = multipleSelection.value.map(item => tableData.value.indexOf(item)).filter(index => index !== -1);
    indexesToRemove.sort((a, b) => b - a).forEach(index => tableData.value.splice(index, 1));
    }) 
  }

  const createRow = () => { 
  let row = {"id": tableData.value.length + 1,"city":null,"area":null,"username":null,"account":null,"age":null,"createtime":"2024-01-01",birthday:"",score:0,info:'',uiEdit:true}
  tableData.value.unshift(row)
}
const handleRowSubmit = (row:any) => { 
  let {isValidate,errorMsg,errorColList} = tableRef.value?.validate(row)
  if(!isValidate){
    row.errorList = errorColList 
    const warnMessage = errorMsg.join('<br/>') 
    ElMessage({
        message: warnMessage,
        grouping: true,
        type: 'warning',
        dangerouslyUseHTMLString: true,
        duration : 3000
      })
    return
  }
  row.errorList = []
  // 开始提交后台
  ElMessage.success('处理成功') 
  row.uiEdit = !row.uiEdit
  nextTick( () => {
    copyTableData =  cloneDeep(tableData.value)
  })
}
 
const handleRowDelete = (row:any) => { 
  ElMessageBox.confirm('是否确认删除数据？')
  .then(function () { 
        let deleteIndex = tableData.value.findIndex(o => o.id === row.id)
        tableData.value.splice(deleteIndex, 1); 
        ElMessage.success('删除成功') 
  })
}

const handleRowCancel = (row:any) => { 
  const index = tableData.value.findIndex( o => row.id === o.id)
  if(index >= 0) {
    let copy =  cloneDeep(copyTableData.find( o => row.id === o.id))
    tableData.value[index] = copy
  }
}
const handleRowEdit = (row:any) => { 
  row.uiEdit = !row.uiEdit
}


const onRowChange = (val,row,column) => { 
  if(column.property === 'city') { // 这里可以做一些清空操作
      row['area'] = ''
  }
}
const multipleSelection = ref([])
const handleSelectionChange = (val) => { 
  multipleSelection.value = val
}
</script>

<template>
  <div>
    <el-config-provider :locale="zhCn"> 
      <h1>element table 动态表格与行编辑</h1>
      <el-button @click="createRow">新增行</el-button>
      <el-button @click="batchDelete">删除行</el-button>
      <DTable ref="tableRef" rowKey="id"  :tableData="tableData" :columns="columns" @onRowChange="onRowChange" @selection-change="handleSelectionChange">  
          <template #operateSlot="{row}"> 
              <el-button v-show="row.uiEdit" type="primary" link @click="handleRowSubmit(row)">保存</el-button> 
              <el-button v-show="!row.uiEdit" type="primary" link @click="handleRowEdit(row)">编辑</el-button> 
              <el-button v-show="row.uiEdit" type="primary" link @click="handleRowCancel(row)" >取消</el-button>
              <el-button v-show="!row.uiEdit" type="primary" link @click="handleRowDelete(row)">删除</el-button>
            </template>    
      </DTable>  
  </el-config-provider>
  </div> 
</template>

<style scoped> 
</style>
