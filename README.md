# vue3-learning
vue3 学习笔记。摘自<https://www.yuque.com/woniuppp/vue3>


## 项目结构  

```
+ 📂 src
  + 📂 assets
    - logo.png          // 资源文件
  + 📂 components  
    - HelloWorld.vue    // 组件
  - App.vue          // Vue 入口
  - index.css        // css 从main.js 导入
  - main.js          // 脚本入口
- index.html         // 网页入口
```  

值得注意的是，这里的`index.html` 需要手动引入`main.js`。而`<script/>` 标签的类型需要设置为`module`。

## 组合式语法  

1. Option 语法  

```vue
<template>
  <h1>{{ msg }}</h1>
  <button @click="count++">count is: {{ count }}</button>
  <p>Edit <code>components/HelloWorld.vue</code> to test hot module replacement.</p>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      count: 0
    }
  }
}
</script>
```

2. Composition 语法  

```vue
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
```

`VSCode` 自带代码提示，较原生`webpack` 好用。  
语法稍有不同，但也容易理解：  
  - `reactive`: 将对象封装成响应式的  
  - `computed`: 返回一个计算属性，同`vue2.x`  
  - `ref`: 将值类型打包成响应式对象  
  - `effect`: 副作用，类似于`watch`

**setup():**  
  `composition` 的入口函数，在`beforeCreate` 之前调用。返回值作为模板渲染的上下文。  

最重要的是可以将上下文的声明封装为不同的函数，让`setup()` 变得非常简洁。  
详解：<https://www.yuque.com/woniuppp/vue3/composition#lHKrC>  

## Fragment  

`vue3` 的模板不要求所有元素在同一根节点下。

```vue
<template>
  <div>1</div>
  <div>2</div>
</template>
```

淦，好帅的快速排序  
```vue
<template>
  <!--递归地渲染-->
  <quick :data="left" v-if="left.length"></quick>
  <li>{{flag}}</li>
  <quick :data="right" v-if="right.length"></quick>
</template>
<script>
export default {
    // 组件的声明和vue2.x 类似
    name:'quick',
    props:['data'],
    setup(props){
        let flag = props.data[0];  // 判断数组中是否还有元素
        let left = []
        let right = []
        // 获取props 中的元素
        props.data.slice(1).forEach(v=>{
            // 大于第一个元素的放在右边；否则放在左边
            v>flag? right.push(v): left.push(v)
        })
        return {left, right, flag}
    }
}
</script>
```

## Teleport  
渲染组件到指定`DOM` 节点。做弹窗比较有用。  

```vue
<template>
  <Teleport to="#foot-container">
      <div>123</div>
  </Teleport>
</template>

</script>
```

## Suspense  

异步组件  

```vue
<Suspense>
  <template #default>
    异步的组件
  </template>
  <!--加载时显示下面的，完成后显示上面的-->
  <template #fallback>
    加载状态的组件
  </template>
</Suspense>
```

```vue
<template>
  <h1>一个异步小组件</h1>
</template>

<script>
function sleep(timeout) {
  return new Promise(resolve => setTimeout(resolve, timeout));
}
export default {
  name: "AsyncComponent",
  props: {
    timeout: {
      type: Number,
      required: true
    }
  },
  // 将setup 函数添加异步标记
  async setup(props) {
    await sleep(props.timeout);
  }
};
</script>
```

对事件的监听也很友好：<https://www.yuque.com/woniuppp/vue3/todomvc>  


## 插槽  

```vue
<!-- default slot -->
<foo v-slot="{ msg }">{{ msg }}</foo>

<!-- named slot -->
<foo>
  <template v-slot:one="{ msg }">
    {{ msg }}
  </template>
  <!-- 简写 -->
  <template #two="{ msg }">
    {{ msg }}
  </template>
</foo>
```

## 全局变量不在挂载在Vue，而是App.vue 上  

```js
import { createApp } from 'vue'
import App from './App.vue'

// 
const app = createApp(App)

app.config.isCustomElement = tag => tag.startsWith('app-')
app.use(/* ... */)
app.mixin(/* ... */)
app.component(/* ... */)
app.directive(/* ... */)

app.config.globalProperties.customProperty = () => {}

app.mount(App, '#app')
```

# 响应式  

可以在任意项目中使用`@vue/reactivity`  

```js
const {ref , effect} = require('@vue/reactivity')

let count = ref(1)

effect(()=>{
    console.log('数据变化了,count是'+count.value)
})
setInterval(()=>{
    count.value++
},1000)
```