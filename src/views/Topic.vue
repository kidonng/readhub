<template>
  <article v-if="topic">
    <h2>{{ spacing(topic.title) }}</h2>
    <time class="meta">{{ format(topic.createdAt) }}</time>
    <div class="summary">
      {{ spacing(topic.summary) }}
    </div>

    <section>
      <h3>媒体报道</h3>
      <ul>
        <li v-for="{ url, title, siteName } in topic.newsArray" :key="url">
          <a :href="url">
            {{ spacing(title) }}
          </a>
          <span class="meta">{{ siteName }}</span>
        </li>
      </ul>
    </section>

    <section v-if="topic.timeline && topic.timeline.topics">
      <h3>
        {{ topic.timeline.commonEntities.length ? '事件追踪' : '相关事件' }}
      </h3>
      <ul class="timeline">
        <li v-for="{ id, title, createdAt } in topic.timeline.topics" :key="id">
          <router-link :to="{ name: 'topic', params: { id } }">
            {{ spacing(title) }}
          </router-link>
          <span class="meta">{{ format(createdAt) }}</span>
        </li>
      </ul>
    </section>
  </article>
</template>

<script setup lang="ts">
import { ref, computed, watchEffect } from 'vue'
import { api } from '../utils/api'
import { router } from '../plugins/router'
import { spacing } from '../plugins/filters'

export const topic = ref<FullTopic>()

watchEffect(async () => {
  const route = computed(() => `topic/${router.currentRoute.value.params.id}`)
  const { timeline, ...data } = await api(route.value).json<FullTopic>()

  // Remove current topic
  if (timeline?.topics) timeline.topics.shift()
  topic.value = { timeline, ...data }

  document.title = `${spacing(data.title)} - Readhub`
})

export { spacing }
export { format } from '../plugins/filters'
</script>

<style lang="stylus" scoped>
@import '../variables'

.summary
  margin-top xxs

li
  & .meta
    display block

    @media (min-width: 768px)
      &
        display initial
        margin-left xxs
</style>
