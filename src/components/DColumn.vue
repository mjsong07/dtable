<template>
  <div :class="{isError: props.row.errorList && props.row.errorList.includes(props.property)}">
    <RenderContent />
  </div>
</template>

<script setup lang="tsx">
import { ElSelect, ElOption, ElInput, ElText, ElDatePicker, ElSelectV2 } from 'element-plus';
import { JSX } from 'vue/jsx-runtime';

interface Option {
  value: number | string;
  label: string;
}

// 定义了表格列配置的属性类型
interface Props {
  column: any, // 表格列的原始数据对象
  type: 'select' | 'input' | 'date'; // 表格列的输入类型，可以是选择框、输入框或者日期选择器
  label: string; // 表格列的标签
  property: string; // 表格列的属性名
  render?: 'component' | 'slot'; // 可选的自定义渲染方式，不传则按员el-table的方式渲染
  options?: Array<Option> | Function; // 选择项数据源，可以是选项数组或者一个返回选项数组的函数
  row: any; // 当前行的数据对象
  modelValue: any; // 当前列的绑定值
  attr?: { // 额外的属性配置对象
    type?: string; // 自定义属性类型
    optionlabel?: string; // 选项标签属性名
    optionValue?: string; // 选项值属性名
    allowCreate?: false; // 是否允许创建新选项（目前仅支持选择框）
    onPropValChange?: Function; // 属性值变化时的回调函数
    precision?: number; // 数字类型时，保留的小数位数
    number?: string; // 是否是数字类型
    maxlength?: number; // 最大长度限制
    format?: string; // 日期格式
    valueFormat?: string; // 绑定值的格式
    max?: number; // 最大值限制
    min?: number; // 最小值限制
  } | undefined;
  readonly?: boolean; // 是否强制只读
}

const props = defineProps<Props>();
const emits = defineEmits(['update:modelValue']);

const regexMap = {
  positive: /[^0-9]/g,
  positiveNegative: /(?<!^)-|[^0-9-]/g,
  float: /[^0-9.]/g,
  floatNegative: /(?<!^)-|[^-0-9.]/g,
};

const processValue = (value: any, props: any) => {
  const regexType = props.attr?.number || 'positive';
  const regex = regexMap[regexType as keyof typeof regexMap];
  value = value.replace(regex, '');

  if (regexType === 'float' || regexType === 'floatNegative') {
    value = value.replace(/^\D*([0-9]\d*\.?\d{0,2})?.*$/, '$1');
  }

  if (props.attr?.max !== undefined && value > props.attr.max) {
    value = props.attr.max;
  }

  if (props.attr?.min !== undefined && value < props.attr.min) {
    value = props.attr.min;
  }

  return value;
};

const onValChange = (value: any) => {
  if (props.attr?.onPropValChange) {
    const obj = props.attr.onPropValChange(value, props);
    if (obj.isError) {
      value = obj.val;
    }
  }
  emits('update:modelValue', value);
};

const onValNumberChange = (value: any) => {
  console.log("onValNumberChange", value)
  value = processValue(value, props);
  emits('update:modelValue', value);
};

const isReadOnly = (props: Props) => {
  if (props.readonly) return true;
  if (props.row.readonlyKeyList?.includes(props.property)) return true;
  return !props.row.uiEdit;
};

const getOptions = (props: Props) => {
  if (typeof props.options === 'function') {
    return props.options(props.row, props.column);
  }
  return props.options || [];
};

const renderSelect = (nowVal: any, readOnly: boolean, options: Option[]) => {
  if (readOnly) {
    const selectedOption = options.find((o: any) => {
      const valueKey = props.attr?.optionValue || 'value';
      return o[valueKey] == nowVal;
    });
    const label = selectedOption?.label || (props.attr?.allowCreate ? nowVal : '');
    return <ElText>{label}</ElText>;
  }

  const selectOptions = options.map((option: Option) => ({
    label: props.attr?.optionlabel ? option[props.attr.optionlabel] : option.label,
    value: props.attr?.optionValue ? option[props.attr.optionValue] : option.value,
  }));

  return options.length > 1000 ? (
    <ElSelectV2
      v-model={props.row[props.property]}
      options={selectOptions}
      onChange={onValChange}
      allow-create={props.attr?.allowCreate}
      filterable
      clearable
    />
  ) : (
    <ElSelect
      v-model={props.row[props.property]}
      onChange={onValChange}
      allow-create={props.attr?.allowCreate}
      default-first-option={props.attr?.allowCreate}
      filterable
      clearable
    >
      {selectOptions.map((option) => (
        <ElOption label={option.label} value={option.value} />
      ))}
    </ElSelect>
  );
};

const renderDate = (nowVal: any, readOnly: boolean) => {
  return readOnly ? (
    <ElText>{nowVal}</ElText>
  ) : (
    <ElDatePicker
      v-model={props.row[props.property]}
      format={props.attr?.format}
      value-format={props.attr?.valueFormat}
      type={props.attr?.type}
    />
  );
};

const renderInput = (nowVal: any, readOnly: boolean) => {
  const formatVal = props.attr?.type === 'number'
    ? formatNum(nowVal, props.attr?.precision || 0)
    : nowVal;

  return readOnly ? (
    <ElText>{formatVal}</ElText>
  ) : (
    <ElInput
      maxlength={props.attr?.maxlength || Infinity}
      v-model={props.row[props.property]}
      onInput={(value: any) => {
        if (props.attr?.type === 'number') {
          onValNumberChange(value);
        } else {
          onValChange(value);
        }
      }}
    />
  );
};

const RenderContent = () => {
  const nowVal = props.row[props.property];
  const readOnly = isReadOnly(props);
  const options = getOptions(props);

  const renderMap: { [key: string]: () => JSX.Element } = {
    select: () => renderSelect(nowVal, readOnly, options),
    date: () => renderDate(nowVal, readOnly),
    input: () => renderInput(nowVal, readOnly),
  };

  return renderMap[props.type] ? renderMap[props.type]() : <ElText>{nowVal}</ElText>;
};

const formatNum = (num: string, fixedDigit: number = 0) => {
  if (num === undefined) {
    return '';
  }
  let numFixed = Number(num).toFixed(fixedDigit);
  let formatNum = `${numFixed}`.replace(/\B(?=(\d{3})+(?!\d))/g, ',');
  return `${formatNum}`;
};
</script>
