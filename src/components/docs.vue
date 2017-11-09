<template>
  <div class="">
    <div class="container">
      <button type="button" name="button" @click="openCNH">Documento</button>
      <button type="button" name="button" @click="openComprovante">Comprovante</button>
    </div>

    <q-modal ref="modal" minimized>
      <q-card v-if="doc.person">
        <q-card-title>
          <h5>{{doc.person.name}}</h5>
        </q-card-title>
        <q-card-main>
          <p v-if="doc.person.name">
            <strong>CPF</strong><br>
            <span class="text-faded">{{doc.person.cpf}}</span>
          </p>
          <p v-if="doc.person.birthdate">
            <strong>Aniversário</strong><br>
            <span class="text-faded">{{doc.person.birthdate}}</span>
          </p>
          <p v-if="doc.person.mothersName">
            <strong>Nome da mãe</strong><br>
            <span class="text-faded">{{doc.person.mothersName}}</span>
          </p>
          <p v-if="doc.person.fathersName">
            <strong>Nome do pai</strong><br>
            <span class="text-faded">{{doc.person.fathersName}}</span>
          </p>
          <p v-if="doc.person.numRegistro">
            <strong>Número de registro</strong><br>
            <span class="text-faded">{{doc.person.numRegistro}}</span>
          </p>
        </q-card-main>
        <q-card-separator />
        <q-card-actions>
          <q-btn flat @click="fecharModal">Fechar</q-btn>
          <q-btn flat @click="encaminhar">Encaminhar</q-btn>
        </q-card-actions>
      </q-card>
    </q-modal>
  </div>
</template>

<script>
import $ from 'jquery'
import {
  Toast,
  QModal,
  QBtn,
  QCard,
  QCardTitle,
  QCardMedia,
  QCardActions,
  QCardSeparator,
  QCardMain,
  QIcon,
  Loading
} from 'quasar'

export default {
  name: 'index',
  data () {
    return {
      modalOpen: false,
      doc: {}
    }
  },
  components: {
    Toast,
    QModal,
    QBtn,
    QCard,
    QCardTitle,
    QCardMedia,
    QCardActions,
    QCardSeparator,
    QCardMain,
    QIcon,
    Loading
  },
  methods: {
    openCNH () {
      navigator.camera.getPicture(this.cnhSuccess, this.cameraError)
    },
    openComprovante () {
      navigator.camera.getPicture(this.comprovanteSuccess, this.cameraError)
    },
    fecharModal () {
      this.doc = {};
      this.$refs.modal.close();
    },
    encaminhar () {
      const mensagem =  [ this.doc.person.name,
                          this.doc.person.cpf,
                          this.doc.person.birthdate,
                          this.doc.person.fathersName,
                          this.doc.person.numRegistro ].join('\n\r')
      window.plugins.socialsharing.share(mensagem)
    },
    cnhSuccess (img) {
      const _this = this
      Loading.show()

      const fd = new FormData();
      window.resolveLocalFileSystemURL(img, function(fileEntry) {
        fileEntry.file(function(file) {
          var reader = new FileReader();
          reader.onloadend = function(e) {
            var imgBlob = new Blob([ this.result ], { type: "image/jpeg" } );
            fd.append('file', imgBlob, file.name);

            $.ajax({
              url: 'http://portal.nxcd.com.br/api/cnh/evaluate/',
              data: fd,
              cache: false,
              contentType: false,
              processData: false,
              method: 'POST',
              mimeType: "multipart/form-data",
              headers: {
                "x-csrf-token": localStorage.getItem('token')
              },
              success: function(data){
                debugger;
                _this.doc = JSON.parse(data);
                _this.$refs.modal.open();
                Toast.create.positive({
                  html: 'Leitura feita com sucesso.'
                })
                Loading.hide()
              },
              error: function(data){
                Toast.create.negative({
                  html: 'Não foi possível fazer a leitura. Tente novamente.'
                })
                Loading.hide()
              }
            });

          };
          reader.readAsArrayBuffer(file);

        }, function(e){console.error(e)});
      }, function(e){console.error(e)});

    },
    comprovanteSuccess (img) {
      const _this = this
      Loading.show()

      const fd = new FormData();
      window.resolveLocalFileSystemURL(img, function(fileEntry) {
        fileEntry.file(function(file) {
          var reader = new FileReader();
          reader.onloadend = function(e) {
            var imgBlob = new Blob([ this.result ], { type: "image/jpeg" } );
            fd.append('file', imgBlob, file.name);
            fd.append('cpfH', '36839854884');

            $.ajax({
              url: 'http://portal.nxcd.com.br/api/proofofresidence/evaluate/',
              data: fd,
              cache: false,
              contentType: false,
              processData: false,
              method: 'POST',
              mimeType: "multipart/form-data",
              headers: {
                "x-csrf-token": localStorage.getItem('token')
              },
              success: function(data){
                debugger;
                _this.doc = JSON.parse(data);
                _this.$refs.modal.open();
                Toast.create.positive({
                  html: 'Leitura feita com sucesso.'
                })
                Loading.hide()
              },
              error: function(data){
                Toast.create.negative({
                  html: 'Não foi possível fazer a leitura. Tente novamente.'
                })
                Loading.hide()
              }
            });

          };
          reader.readAsArrayBuffer(file);

        }, function(e){console.error(e)});
      }, function(e){console.error(e)});

    }
  }
}
</script>

<style lang="stylus">
.container
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background-color: tomato;

  button
    background-color: white;
    border: 0;
    padding: 30px;
    font-size: 20px;
    font-weight: bold;
    color: tomato;
    min-width: 200px
    margin: 20px

    &:focus,
    &:active
      background-color: #EDEDED;

</style>
