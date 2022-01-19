<template>
  <VirtualList :listData="listData" :itemSize="68" v-slot="slotItem">
    <div :item="slotItem.item">{{slotItem.item}}</div>
  </VirtualList>
</template>

<script>
import VirtualList from './components/VirtualList.vue'
import { reactive, toRefs } from 'vue';

export default {
  name: 'App',
  components: {
    VirtualList
  },
  setup() {
    const state = reactive({
      listData: []
    })
    state.listData = Array.from(Array(500).keys())
    return {
      ...toRefs(state)
    }
  }
}
</script>

<style>
html{
  height: 100%;
}
body{
  height: 100%;
  margin:0;
}
#app{
  height:100%;
}
</style>
