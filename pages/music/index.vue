<script>
export default {
  mounted() {
    console.log(this.$route)
  },
  methods: {
    naviageTo() {
      this.$router.push({
        name: 'music-genre',
        params: {
          genre: 'pop',
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
    halaman music
    <button @click="naviageTo">navigate</button>
  </div>
</template>
