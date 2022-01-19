<template>
  <div ref="list" class="infinite-list-container" @scroll="handleScroll($event)">
    <div class="infinite-list-phantom" :style="{ height: listHeight + 'px' }"></div>
    <div ref="content" class="infinite-list">
      <div ref="items"
        class="infinite-list-item"
        v-for="item in visibleData"
        :key="item"
        :style="{height: itemSize + 'px'}"
      >
        <slot ref="slot" :item="item"></slot>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, reactive, computed, toRefs, ref } from 'vue';

export default {
  name: 'VirtualList',
  props: {
    listData:{
      type:Array,
      default:()=>[]
    },
    //每项高度
    itemSize: {
      type: Number,
      default:200
    },
    bufferScale:{
      type:Number,
      default:1
    }
  },
  setup(props) {
    const state = reactive({
      screenHeight:0,
      //偏移量
      startOffset:0,
      //起始索引
      start:0,
      //结束索引
      end:null,
    })
    //获取list Dom
    const list = ref(null)
    //获取content Dom
    const content = ref(null)
    //列表总高度
    const listHeight = computed(() => { return props.itemSize * props.listData.length})
    //可显示的列表项数
    const visibleCount = computed(() => { return Math.ceil(state.screenHeight / props.itemSize)})
    //偏移量对应的style
    const getTransform = computed(() => { return `translate3d(0,${state.startOffset}px,0)`})
    //缓冲量阀值
    const threshold = computed(() => { return props.bufferScale * visibleCount.value})

    //获取真实显示列表数据
    // 上方缓冲数据量
    const aboveCount = computed(() => { return Math.min(state.start, threshold.value)})
    // 下方缓冲数据量
    const belowCount = computed(() => { return Math.min(props.listData.length - state.end, threshold.value)})
    const visibleData = computed(() => {
      let start = state.start - aboveCount.value
      let end = state.end + belowCount.value
      return props.listData.slice(start, end)
    })

    onMounted(() => {
      state.screenHeight = document.body.clientHeight
      state.start = 0
      state.end = state.start + visibleCount.value
    })

    function handleScroll () {
      //当前滚动位置
      let scrollTop = list.value.scrollTop
       //此时的开始索引
      state.start = Math.floor(scrollTop / props.itemSize);
      //此时的结束索引
      state.end = state.start + visibleCount.value
      //此时的偏移量
      if (state.start >= threshold.value) {
        state.startOffset = scrollTop - (threshold.value * props.itemSize) - (scrollTop % props.itemSize)
        content.value.style.transform = `translate3d(0,${state.startOffset}px,0)`
      } else {
        content.value.style.transform = `translate3d(0,0,0)`
      }
    }

    return {
      ...toRefs(state),
      list,
      content,
      listHeight,
      getTransform,
      visibleData,
      handleScroll,
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.infinite-list-container {
  height: 100%;
  overflow: auto;
  position: relative;
  -webkit-overflow-scrolling: touch;
}
.infinite-list-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}
.infinite-list {
  left: 0;
  right: 0;
  top: 0;
  position: absolute;
  text-align: center;
}
.infinite-list-item {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px;
  color: #555;
  box-sizing: border-box;
  border-bottom: 1px solid #999;
}
</style>
