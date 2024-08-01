<template>
  <el-table :data="props.tableData" :row-key="props.rowKey">
    <el-table-column
      v-for="column in props.columns"
      :key="column.property"
      :label="column.label"
      :prop="column.property"
      :align="column.align ? column.align : 'center'"
      :type="column.type"
      :width="column.width ? column.width : 100"
      :fixed="column.fixed ? column.fixed : false"
    >  
    <template #header>
      <span v-if="column.validate" class="validateTips">*</span>
      <span class="title">{{column.label}}</span>
    </template>  
    <template v-if="column.render === 'component' " #default="scope"> 
      <DColumn  
        :column="column"
        :type="column.type"
        :label="column.label"
        :property="column.property"
        :options="column.options" 
        :attr="column.attr ? column.attr  : {}"
        :readonly="column.readonly ? column.readonly  : false" 
        :modelValue="scope.row[column.property]"
        :row="scope.row"
        @update:modelValue="($event) => {onChange($event,scope.row,column)} " 
      >
      </DColumn> 
    </template>
    <template v-if="column.render === 'slot'"  #default="scope">
      <slot  :name="column.slotName" :row="scope.row" ></slot> 
    </template> 
    </el-table-column>
</el-table>


</template>
  
<script setup lang="tsx"> 
import DColumn from './DColumn.vue'; 
  interface Props { 
    rowKey: string;
    tableData: Array<any>;
    columns: Array<any>;  
      rules?: any;   
  }
const emits = defineEmits(['onRowChange']);
 const onChange = (val,row,column) => { 
   row[column.property] = val
   emits('onRowChange',val,row,column)
 }

const props = defineProps<Props>();
//校验当前行
const validate = (row:any,ignore:Array<string> = []) => { 
  const isIgnore = (val:string) => ignore.includes(val)
  const isEmpty = (val:any) =>  val === '' || val == null
  const isSmallZero = (val:any) =>  val  < 0
  const isNumberInput = (col:any) => (col.attr || {}).type === 'number'; 
  const validateColumn = (col:any) => { 
    let value = row[col.property]
    if(isNumberInput(col) && ['positive','float'].includes(col.attr.number) ) {
      if(isSmallZero(value)) { 
        return `${col.label}不能小于0`
      } else {
        return isEmpty(value) ? `${col.label}不能为空` : undefined
      }
    } 
    return !value ? `${col.label}不能为空` : undefined
  }
  const  validateObj = props.columns.filter(col => !isIgnore(col.property) && col.validate).reduce((obj,col) => {
    let msg = validateColumn(col) 
    if(msg) {
      obj.isValidate = false 
      obj.errorMsg.push(msg) 
      obj.errorColList.push(col.property) 
    }
    return obj
  },{isValidate:true,errorMsg:[],errorColList:[]}) 
  return validateObj
}

defineExpose({
  validate
})

  </script>