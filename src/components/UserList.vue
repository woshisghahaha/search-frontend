<template>
  <!-- 数据源来自HomeView.vue通过myAxios取到的后端数据 -->
  <!--
  这个属性使用Vue.js的动态绑定语法，
  将props.userList作为数据源传递给列表组件。
  props.userList是从父组件传递进来的属性值。
  -->
  <a-list
    item-layout="horizontal"
    :grid="{ gutter: 16, xs: 1, sm: 2, md: 4, lg: 4, xl: 6, xxl: 3 }"
    :data-source="userList"
  >
    <template #renderItem="{ item }">
      <a-list-item>
        <a-card hoverable style="width: 240px">
          <template #cover>
            <img alt="example" :src="cutePic" />
          </template>
          <!-- item里的数据与传进来的userList里的user有什么数据有关 -->
          <a-card-meta :title="item.userName">
            <template #description>{{ item.userProfile }}</template>
          </a-card-meta>
        </a-card>
      </a-list-item>
    </template>
  </a-list>
</template>

<script lang="ts">
//分页
import { defineComponent, toRefs } from "vue";
//import cutePic from "../assets/cutePic.jpg";
export default defineComponent({
  props: {
    userList: {
      type: Array,
      required: true,
      default: () => [],
    },
  },
  setup(props) {
    return {
      ...toRefs(props),
      cutePic: require("../assets/gege.jpg"), //在 setup 函数中，记得将 cutePic 变量添加到了返回的对象中。
    };
  },
});
</script>

<style scoped>
.cutePic {
  width: 100px;
}
</style>
