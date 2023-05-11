<template>
  <div style="display: flex; gap: 10px">
    <div style="flex: 1">
      ({{ updateParts.viewStartIdx }} - [{{ updateParts.visibleStartIdx }} -
      {{ updateParts.visibleEndIdx }}] - {{ updateParts.viewEndIdx }})

      {{ isLoading }}
    </div>
    <div ref="el" id="app1">
      <div v-for="post in posts" :key="post.id">
        <PostCard :item="post" />
      </div>
      <p style="text-align: center" v-if="isLoading">Loading...</p>
      <!-- <DynamicScroller
        v-if="posts.length"
        keyField="id"
        :emit-update="true"
        :items="posts"
        :min-item-size="200"
        @resize="onResize"
        @update="onUpdate"
      >
        <template v-slot="{ item, index, active }">
          <DynamicScrollerItem
            :data-index="index"
            :data-active="active"
            :size-dependencies="[item.body]"
            :item="item"
            :active="active"
            tag="article"
          >
            <PostCard :item="item" />
          </DynamicScrollerItem>
        </template>
      </DynamicScroller> -->
    </div>
  </div>
</template>

<script>
import "vue-virtual-scroller/dist/vue-virtual-scroller.css";

import { ref, onMounted, computed, nextTick } from "vue";
import { DynamicScrollerItem, DynamicScroller } from "vue-virtual-scroller";

import { useInfiniteScroll } from "@vueuse/core";
import { debounce } from "lodash";

import PostCard from "./components/PostCard.vue";
export default {
  name: "App",
  components: {
    PostCard,
    DynamicScrollerItem,
    DynamicScroller,
  },
  setup() {
    const posts = ref([]);
    const el = ref(null);

    const updateParts = ref({
      viewStartIdx: 0,
      viewEndIdx: 0,
      visibleStartIdx: 0,
      visibleEndIdx: 0,
    });

    const pagination = ref({
      page: 1,
      limit: 10,
      last_page: 10,
    });

    const url = computed(
      () =>
        `https://jsonplaceholder.typicode.com/posts?_page=${pagination.value.page}&_limit=${pagination.value.limit}`
    );

    const isLoading = ref(true);

    function loadMore() {
      isLoading.value = true;
      return fetch(url.value)
        .then((blob) => blob.json())
        .then((res) => {
          posts.value = [...posts.value, ...res];
          isLoading.value = false;
          pagination.value.page++;
        });
    }

    useInfiniteScroll(
      el,
      () => {
        if (pagination.value.last_page === pagination.value.page) return;

        else if (pagination.value.page >= 1) {
          return loadMore();
        } 
      },
      {
        distance: 100,
        interval: 2000,
      }
    );

    function onResize() {
      console.log("resize");
    }

    function onUpdate(
      viewStartIndex,
      viewEndIndex,
      visibleStartIndex,
      visibleEndIndex
    ) {
      console.log({
        viewStartIndex,
        viewEndIndex,
        visibleStartIndex,
        visibleEndIndex,
      });
      updateParts.value.viewStartIdx = viewStartIndex;
      updateParts.value.viewEndIdx = viewEndIndex;
      updateParts.value.visibleStartIdx = visibleStartIndex;
      updateParts.value.visibleEndIdx = visibleEndIndex;
    }

    return {
      posts,
      el,
      pagination,
      updateParts,
      onResize,
      onUpdate,
      isLoading,
    };
  },
};
</script>

<style>
body {
  overflow: hidden;
}

#app1 {
  height: 100vh;
  width: 400px;
  overflow-y: auto;
  margin-left: auto;
}
</style>
