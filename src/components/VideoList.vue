<template>
  <div class="video-list">
    <a-list :grid="{ gutter: 16, column: 5 }" :data-source="videoList">
      <template #renderItem="{ item }">
        <a-card :hoverable="true" style="width: 260px">
          <template #cover>
            <a-image :height="220" :src="item.url" />
          </template>
          <a-card-meta>
            <template #description>
              <a-tooltip :title="item.title">
                <a :href="item.vurl" target="_blank" rel="noreferrer">
                  <span v-html="item.title"></span>
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
    videoList: {
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
