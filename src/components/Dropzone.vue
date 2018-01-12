<template>
  <div class="dropzone">
    <div class="info">
      Drop files here or click to upload
    </div>
    <input @change="e => handleFile(e)" class="input" type="file">
  </div>
</template>

<script>
import firebase from '@/fire'

export default {
  name: 'Dropzone',
  data () {
    return {
    }
  },
  methods: {
    handleFile (e) {
      const file = e.target.files[0]
      firebase.storage().ref('files/' + file.name).put(file)
      console.log(e)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.dropzone {
  position: relative;
  margin: auto;
  width: 80%;
  height: 300px;
  background-color: white;
  border-style: dashed;
  border-color: orange;
  border-width: medium;
  border-radius: 4px;
  transition: all 400ms;
}

.dropzone:hover {
  background-color: lightgray;
}

.info {
  position: absolute;
  left: 0;
  top: 40%;
  width: 100%;
  font-size: 1.5em;
  font-weight: bold;
  text-align: center;
}

.input {
  cursor: pointer;
  width: 100%;
  height: 100%;
  opacity: 0;
}
</style>
