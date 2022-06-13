<script>
export default {
  name: 'Errors',
  props: ['errors']
}
</script>

<template>
  <span>{{ errors.length }}</span>
  <i-popover placement="bottom" trigger="hover">
    <i-icon name="ink-danger" class="_color:danger" />
    <template #body>
      <div v-for="error in errors" :key="error">{{ error }}<br><br></div>
    </template>
  </i-popover>
</template>
