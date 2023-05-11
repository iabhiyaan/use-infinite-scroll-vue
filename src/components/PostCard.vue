<template>
  <div class="post-card">
    <h4 style="margin: 5px 0;">{{ item.title }}</h4>
    <p>{{ item.body }}</p>
  </div>
</template>

<script>
export default {
  props: {
    item: Object,
  },
};
</script>

<style>
.post-card {
  padding: 15px 10px;
  border-bottom: 1px solid #ccc;
  /* margin: 10px; */
  /* display: inline-block; */
}
</style>
