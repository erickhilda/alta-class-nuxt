<script>
export default {
  computed: {
    id() {
      return this.$route.params.id
    },
  },
  mounted() {
    console.log(this.$route)
  },
}
</script>

<template>
  <div>{{ id }}</div>
</template>
