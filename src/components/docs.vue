<template>
  <div>
    <div class="container">
      <button type="button" name="button" @click="openDocs">Documento</button>
      <button type="button" name="button" @click="openComprovante">Comprovante</button>
    </div>

    <q-modal ref="modal" :content-css="{minWidth: '80vw', minHeight: '80vh'}">
      <q-modal-layout>
        <q-toolbar slot="header">
          <q-btn flat @click="fecharModal"> <q-icon name="arrow_back" /></q-btn>
        </q-toolbar>

        <div class="layout-padding">
          <q-card v-for="doc in docs" :key="doc.name || doc.person.name">
            <div v-if="doc.person">
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
                <q-btn flat @click="encaminhar(doc)">Encaminhar</q-btn>
              </q-card-actions>
            </div>

            <div v-else>
              <q-card-title>
                <h5>{{doc.name}} <small>({{doc.familyMemberType}})</small></h5>
              </q-card-title>
              <q-card-main>
                <p>
                  <strong>CPF</strong><br>
                  <span class="text-faded">{{doc.cpf}}</span>
                </p>
                <p>
                  <strong>Endereço</strong><br>
                  <span class="text-faded">
                    {{doc.addressType}} {{doc.address}}, {{doc.addressNumber}} <br>
                    {{doc.zipcode}} - {{doc.neighborhood}} - {{doc.city}} - {{doc.state}}
                  </span>
                </p>
              </q-card-main>
              <q-card-separator />
              <q-card-actions>
                <q-btn flat @click="encaminhar(doc)">Encaminhar</q-btn>
              </q-card-actions>
            </div>
          </q-card>
        </div>

      </q-modal-layout>
    </q-modal>


  </div>
</template>

<script>
import $ from 'jquery'
import {
  QModal,
  QModalLayout,
  QToolbar,
  QBtn,
  QCard,
  QCardTitle,
  QCardMedia,
  QCardActions,
  QCardSeparator,
  QCardMain,
  QIcon,
  Loading,
  Toast,
  Dialog
} from 'quasar'

export default {
  name: 'index',
  data () {
    return {
      modalOpen: false,
      docs: [],
      type: null,
      urlDocs: 'http://portal.nxcd.com.br/api/cnh/evaluate/',
      urlComprovante: 'http://portal.nxcd.com.br/api/proofofresidence/evaluate/',
      cpfComprovante: ''
    }
  },
  components: {
    QModal,
    QModalLayout,
    QToolbar,
    QBtn,
    QCard,
    QCardTitle,
    QCardMedia,
    QCardActions,
    QCardSeparator,
    QCardMain,
    QIcon,
    Loading,
    Toast,
    Dialog
  },
  methods: {
    openDocs () {
      navigator.camera.getPicture(this.cameraDocs)
    },

    openComprovante () {
      const _this = this

      Dialog.create({
        title: 'Comprovante',
        message: 'Insira seu cpf para validação do comprovante.',
        form: {
          cpf: {
            type: 'number',
            label: 'CPF (Somente números)',
            model: ''
          }
        },
        buttons: [
          'Cancel',
          {
            label: 'Ok',
            handler (data) {
              _this.cpfComprovante = data.cpf;
              navigator.camera.getPicture(_this.cameraComprovante);
            }
          }
        ]
      })

    },

    cameraDocs (img) {
      const _this = this
      Loading.show()

      this.getImagem(img, function(fd) {
        _this.leituraImagem(_this.urlDocs, fd, function(data) {
          debugger;
          _this.docs = [JSON.parse(data)];
          _this.$refs.modal.open();
          Toast.create.positive({
            html: 'Leitura feita com sucesso.'
          })
          Loading.hide();
        })
      })
    },

    cameraComprovante (img) {
      const _this = this
      Loading.show()

      this.getImagem(img, function(fd) {
        fd.append('cpfH', _this.cpfComprovante);

        _this.leituraImagem(_this.urlComprovante, fd, function(data) {
          debugger;
          _this.docs = JSON.parse(data).addresses;
          _this.$refs.modal.open();
          Toast.create.positive({
            html: 'Leitura feita com sucesso.'
          })
          Loading.hide();
        })
      })

    },

    getImagem (img, callback) {
      const fd = new FormData();
      window.resolveLocalFileSystemURL(img, function(fileEntry) {
        fileEntry.file(function(file) {
          var reader = new FileReader();
          reader.onloadend = function(e) {

            var imgBlob = new Blob([ this.result ], { type: "image/jpeg" } );
            fd.append('file', imgBlob, file.name);

            callback(fd)

          };
          reader.readAsArrayBuffer(file);

        }, function(e){console.error(e)});
      }, function(e){console.error(e)});
    },

    leituraImagem (url, data, callback) {
      $.ajax({
        url: url,
        data: data,
        cache: false,
        contentType: false,
        processData: false,
        method: 'POST',
        mimeType: "multipart/form-data",
        headers: {
          "x-csrf-token": localStorage.getItem('token')
        },
        success: function(data){
            callback(data);
        },
        error: function(data){
          Toast.create.negative({
            html: 'Não foi possível fazer a leitura. Tente novamente.'
          })
          Loading.hide()
        }
      });
    },

    fecharModal () {
      this.doc = {};
      this.$refs.modal.close();
    },

    encaminhar (doc) {
      let mensagem = []
      if(doc.person) {
        mensagem = [doc.person.name,
                    doc.person.cpf,
                    doc.person.birthdate,
                    doc.person.fathersName,
                    doc.person.numRegistro].join('\n\r')
      }
      else {
        const endereco =  [[doc.addressType, doc.address, doc.addressNumber].join(' '),
                          [doc.zipcode, doc.neighborhood, doc.city, doc.state].join(' - ')].join('\n\r')
        mensagem = [doc.name,
                    doc.cpf,
                    endereco].join('\n\r')
      }

      window.plugins.socialsharing.share(mensagem)
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

h5
  text-transform: capitalize;

.q-toolbar
  background-color: tomato;

.layout-header
  box-shadow: none;

</style>
