<script>
export default {
  mounted() {
    console.log(this.$route)
  },
  methods: {
    naviageTo() {
      this.$router.push({
        name: '"music-genre-title"',
        params: {
          title: 'dewa',
        },
        query: {
          limit: 10,
        },
      })
    },
  },
}
</script>

<template>
  <div>
    halaman music/_genre
    <button @click="naviageTo">navigate</button>
  </div>
</template>
