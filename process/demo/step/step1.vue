<template>
  <div class="box">
    <Button @click="jump('step3')">jump to step3</Button>
    <Button @click="steps[0].next = !steps[0].next">toggle next</Button>
    <Button @click="steps[0].disabled = !steps[0].disabled">toggle enable</Button>
    <Input v-model="modal.step1.title[0].name" placeholder="全局数据源" style="width:200px"/>
    <Input v-model="modal.name" placeholder="全局数据源1" style="width:200px"/>
  </div>
</template>
<script>
export default {
  name: 'step1',
  inject: ['modal','jump', 'steps']
}
</script>
<style lang="less" scoped>
  .box {
    border: 1px solid #ccc;
    height: 400px;
  }
</style>