<template>
  <!-- 数据源来自HomeView.vue通过myAxios取到的后端数据 -->
  <!--
  这个属性使用Vue.js的动态绑定语法，
  将props.postList作为数据源传递给列表组件。
  props.postList是从父组件传递进来的属性值。
  -->
  <a-list item-layout="horizontal" size="large" :data-source="postList">
    <template #renderItem="{ item }">
      <a-list-item>
        <template #actions>
          <a-button type="primary">more</a-button>
        </template>
        <!-- 使用Vue.js的动态绑定语法，将item.content绑定到:description属性上，实现动态展示。 -->
        <a-list-item-meta>
          <template #title>
            <a
              :href="`https://www.code-nav.cn/post/${item.id}`"
              target="_blank"
              rel="noreferrer"
            >
              <p v-html="item?.title"></p>
            </a>
          </template>
          <template #avatar>
            <!-- 使用:src属性将cutePic作为头像的源。 -->
            <a-avatar :src="cutePic" />
          </template>
          <template #description>
            <p
              v-html="
                item?.content ? item.content.substring(0, 200) : item.content
              "
            />
          </template>
        </a-list-item-meta>
        <!--标签-->
        <div class="tagList">
          <a-tag v-for="(tag, index) in item?.tagList" :key="index"
            >{{ tag }}
          </a-tag>
        </div>
      </a-list-item>
    </template>
  </a-list>
</template>

<script lang="ts">
//分页
import { defineComponent, toRefs } from "vue";
export default defineComponent({
  props: {
    postList: {
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
