<script setup lang="ts">
const params = useRoute().params;

const { data: blog } = await useAsyncData(() =>
  queryCollection("content").path(`/blogs/${params.slug}`).first()
);

useSeoMeta({
  title: blog.value?.title,
  description: blog.value?.description,
});
</script>

<template>
  <ContentRenderer v-if="blog" :value="blog" />
  <div v-else>Blog not found</div>
</template>
