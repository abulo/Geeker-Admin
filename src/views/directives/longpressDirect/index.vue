<template>
  <div class="card content-box">
    <span class="text">长按指令 🍇🍇🍇🍓🍓🍓</span>
    <el-button v-longpress="longpress" type="primary"> 长按2秒触发事件 </el-button>
  </div>
</template>

<script setup lang="ts">
defineOptions({ name: 'LongpressDirect' })
import { ElMessage } from 'element-plus'
const longpress = () => {
  ElMessage.success('长按事件触发成功 🎉🎉🎉')
}
</script>

<style scoped lang="scss">
@use './index';
</style>
