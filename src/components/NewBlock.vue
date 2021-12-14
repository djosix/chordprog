<template lang="pug">
.new-block
  button(type="button") new
  button(type="button") duplicate
</template>

<script>
export default {
  name: 'Template',
  props: {
    prop: String
  }
}
</script>

<style scoped>
.new-block {
  display: inline-block;
  border: 2px solid black;
  border-radius: 0.4em;
  padding: 0.5em;
  margin-left: 0.2em;
  margin-right: 0.2em;
}
</style>
