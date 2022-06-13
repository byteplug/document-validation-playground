<script>
export default {
  name: 'Warnings',
  props: ['warnings']
}
</script>

<template>
  <span>{{ warnings.length }}</span>
  <i-popover placement="bottom" trigger="hover">
    <i-icon name="ink-warning" class="_color:warning" />
    <template #body>
      <div v-for="warning in warnings" :key="warning">{{ warning }}<br><br></div>
    </template>
  </i-popover>
</template>
