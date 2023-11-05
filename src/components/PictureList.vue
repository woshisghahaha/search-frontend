<template>
  <div class="picture-list">
    <a-list :grid="{ gutter: 16, column: 5 }" :data-source="picList">
      <template #renderItem="{ item }">
        <a-card :hoverable="true" style="width: 220px">
          <template #cover>
            <a-image :height="220" :src="item.url" :alt="item.title" />
          </template>
          <a-card-meta>
            <template #description>
              <a-tooltip :title="item.title">
                <a :href="item.url" target="_blank" rel="noreferrer">
                  <span v-html="item.title" />
                </a>
              </a-tooltip>
            </template>
          </a-card-meta>
        </a-card>
      </template>
    </a-list>
  </div>
</template>

<script lang="ts">
//分页
import { defineComponent, toRefs } from "vue";

export default defineComponent({
  props: {
    picList: {
      type: Array,
      required: true,
      default: () => [],
    },
  },
  setup(props) {
    return {
      ...toRefs(props),
    };
  },
});
</script>

<style scoped></style>
