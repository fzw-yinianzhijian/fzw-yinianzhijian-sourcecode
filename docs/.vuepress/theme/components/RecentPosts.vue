<script setup lang="ts">
import { computed } from 'vue'
import { useLocalePostList } from 'vuepress-theme-plume/composables'

const posts = useLocalePostList()
const recentPosts = computed(() => posts.value
  .filter(post => !post.draft && !post.tags?.includes('Pinned'))
  // Plume normalizes createTime to YYYY/MM/DD HH:mm:ss.
  // Sort a copy so this section never changes the main blog list's order.
  .sort((a, b) => b.createTime.localeCompare(a.createTime) || a.path.localeCompare(b.path))
  .slice(0, 2))
</script>

<template>
  <div class="recent-posts">
    <LinkCard
      v-for="post in recentPosts"
      :key="post.path"
      :title="post.title"
      :href="post.path"
    >
      发布于 {{ post.createTime.slice(0, 10).replaceAll('/', '-') }}
    </LinkCard>
    <p v-if="recentPosts.length === 0">新故事还在路上，过会儿再来坐坐。</p>
  </div>
</template>
