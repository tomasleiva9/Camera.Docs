<template>
  <div class="container">
    <button type="button" name="button" @click="openCamera">abrir câmera</button>
  </div>
</template>

<script>

export default {
  name: 'index',
  data () {
    return {
    }
  },
  computed: {
  },
  methods: {
    openCamera () {
      navigator.camera.getPicture(this.cameraSuccess, this.cameraError)
    },
    cameraSuccess () {
      alert('Enviada com sucesso!')
    },
    cameraError () {
      alert('error')
    }
  },
  mounted () {

  },
  beforeDestroy () {

  }
}
</script>

<style lang="stylus">
.container
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background-color: tomato

  button
    background-color: white;
    border: 0;
    padding: 30px;
    font-size: 20px;
    font-weight: bold;
    color: tomato;

    &:focus,
    &:active
      background-color: #EDEDED;

</style>
