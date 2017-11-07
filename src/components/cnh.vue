<template>
  <div class="container">
    <button type="button" name="button" @click="openCamera">abrir câmera</button>
  </div>
</template>

<script>
import $ from 'jquery'

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
    cameraSuccess (img) {

      var fd = new FormData();
      window.resolveLocalFileSystemURL(img, function(fileEntry) {
        fileEntry.file(function(file) {
          fd.append('file', file);
          debugger
          $.ajax({
            url: 'http://portal.nxcd.com.br/api/cnh/evaluate/',
            data: fd,
            cache: false,
            contentType: false,
            processData: false,
            dataType: 'json',
            method: 'POST',
            headers: {
              "x-csrf-token": localStorage.getItem('token')
            },
            success: function(data){
              debugger
              console.log(data)
              alert('Enviada com sucesso!')
            }
          });

        }, function(e){console.error(e)});
      }, function(e){console.error(e)});

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
