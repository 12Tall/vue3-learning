<template>
  <h1>{{state.count}} * 2 = {{double}}</h1>
  <h1>num = {{num}}</h1>
  <button @click="add">累加</button>
</template>

<script>
import { reactive, ref, onMounted, computed } from "vue";
export default {
  setup() {
    // 声明变量：reactive 表示对象；ref 封装值类型
    const state = reactive({ count: 1 }),
      num = ref(1);

    // 定义函数
    function add() {
      state.count++;
      // 仅可操作ref 封装的对象的value 值
      num.value += 10;
    }

    // 组件的生命周期测试
    onMounted(() => {
      console.log("mounted");
    });

    console.log("created");

    // 封装computed 字段
    const double = computed(() => state.count * 2);

    // 导出定义
    return { state, add, double,num };
  },
};
</script>
