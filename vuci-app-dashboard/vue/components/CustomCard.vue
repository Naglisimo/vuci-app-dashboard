<template>
  <a-card v-if="item.visible">
    <template #title>
      <h2>{{ item.name }}</h2>
    </template>
    <a-list size="small" item-layout="horozontal" class="truncate">
      <a-list-item
        v-for="(note, index) in item.description"
        :key="index"
        :title="note.value"
      >
        <a-list-item-meta >
        <template #description >
          <div class="truncate">{{ note.value }}</div>
        </template>
          <template #title>
            <p>{{ note.title }}</p>
          </template>
        </a-list-item-meta>
      </a-list-item>
    </a-list>
  </a-card>
</template>

<script>
export default {
  name: 'CustomCard',
  props: ['item']
}
</script>

<style scoped>
.truncate {
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
}
</style>
