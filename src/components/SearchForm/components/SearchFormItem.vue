<template>
  <component
    :is="column.search?.render ?? `el-${column.search?.el}`"
    v-bind="{ ...handleSearchProps, ...attrs, ...placeholder, searchParam: _searchParam, clearable }"
    v-model.trim="_searchParam[column.search?.key ?? handleProp(column.prop!)]"
    :data="column.search?.el === 'tree-select' ? columnEnum : []"
    :options="['cascader', 'select-v2'].includes(column.search?.el!) ? columnEnum : []"
  >
    <template v-if="column.search?.el === 'cascader'" #default="{ data }">
      <span>{{ data[fieldNames.label] }}</span>
    </template>
    <template v-if="column.search?.el === 'select'">
      <component
        :is="`el-option`"
        v-for="(col, index) in columnEnum"
        :key="index"
        :label="col[fieldNames.label]"
        :value="col[fieldNames.value]"
      />
    </template>
    <slot v-else></slot>
  </component>
</template>

<script setup lang="ts">
defineOptions({ name: 'SearchFormItem' })
import { handleProp } from '@/utils'
import type { ColumnProps } from '@/components/ProTable/interface'

interface SearchFormItem {
  column: ColumnProps
  searchParam: { [key: string]: any }
  attrs?: { [key: string]: any }
}
const props = defineProps<SearchFormItem>()

// Re receive SearchParam
const _searchParam = computed(() => props.searchParam)

// 判断 fieldNames 设置 label && value && children 的 key 值
const fieldNames = computed(() => {
  return {
    label: props.column.fieldNames?.label ?? 'label',
    value: props.column.fieldNames?.value ?? 'value',
    children: props.column.fieldNames?.children ?? 'children',
  }
})

// 接收 enumMap (el 为 select-v2 需单独处理 enumData)
const enumMap = inject('enumMap', ref(new Map()))
const columnEnum = computed(() => {
  let enumData = enumMap.value.get(props.column.prop)
  if (!enumData) {
    return []
  }
  if (props.column.search?.el === 'select-v2' && props.column.fieldNames) {
    enumData = enumData.map((item: { [key: string]: any }) => {
      return { ...item, label: item[fieldNames.value.label], value: item[fieldNames.value.value] }
    })
  }
  return enumData
})

// 处理透传的 searchProps (el 为 tree-select、cascader 的时候需要给下默认 label && value && children)
const handleSearchProps = computed(() => {
  const label = fieldNames.value.label
  const value = fieldNames.value.value
  const children = fieldNames.value.children
  const searchEl = props.column.search?.el
  let searchProps = props.column.search?.attrs ?? {}
  if (searchEl === 'tree-select') {
    searchProps = { ...searchProps, props: { ...searchProps, label, children }, nodeKey: value }
  }
  if (searchEl === 'cascader') {
    searchProps = { ...searchProps, props: { ...searchProps, label, value, children } }
  }
  return searchProps
})

// 处理默认 placeholder
const placeholder = computed(() => {
  const search = props.column.search
  if (['datetimerange', 'daterange', 'monthrange'].includes(search?.attrs?.type) || search?.attrs?.isRange) {
    return {
      rangeSeparator: search?.attrs?.rangeSeparator ?? '至',
      startPlaceholder: search?.attrs?.startPlaceholder ?? '开始时间',
      endPlaceholder: search?.attrs?.endPlaceholder ?? '结束时间',
    }
  }
  const placeholder =
    search?.attrs?.placeholder ?? (search?.el?.includes('input') ? '请输入' : '请选择') + ' ' + props.column.label
  return { placeholder }
})

// 是否有清除按钮 (当搜索项有默认值时，清除按钮不显示)
const clearable = computed(() => {
  const search = props.column.search
  return search?.attrs?.clearable ?? (search?.defaultValue == null || search?.defaultValue == undefined)
})
</script>
