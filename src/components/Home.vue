<template>
  <div class="hello">
    <h1 class="title">Print me</h1>
    <dropzone></dropzone>
  </div>
</template>

<script>
import Dropzone from '@/components/Dropzone'

export default {
  name: 'Home',
  components: { Dropzone },
  data () {
    return {

    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  margin: auto;
}

.title {
  text-align: center;
}
</style>
