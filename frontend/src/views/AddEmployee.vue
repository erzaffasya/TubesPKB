<!-- eslint-disable -->
<template>
  <v-container id="attd-list" fluid tag="section">
    <v-row justify="center">
      <v-col 
        cols="12"
        md="12">
        <base-material-card
          icon="mdi-account-plus"
          title="Tambah Mahasiswa"
          class="px-5 py-3"
        > 
          <v-form>
            <v-container class="py-0">
              <v-row>
                <v-col
                  cols="12"
                  md="6">
                  <v-text-field
                    v-model="nik"
                    label="NIM Mahasiswa"
                    type="number"
                    outlined
                  />
                </v-col>
                <v-col
                  cols="12"
                  md="6">
                  <v-text-field
                    v-model="nama"
                    label="Nama Lengkap"
                    outlined
                  />
                </v-col>
              </v-row>
              <v-row>
                <v-col cols="12" md="12">
                  <div v-if="isCameraOpen" class="camera-box" align="center">  
                    <video ref="camera" :width="450" :height="337.5" autoplay></video>
                    <canvas class="ml-3" v-show="isPhotoTaken" id="photoTaken" ref="canvas" :width="450" :height="337.5"></canvas>
                  </div>
                  <div class="camera-button" align="center">
                    <v-btn :disabled="!formFilled || viaLive" class="blue" @click="toggleFormUpload">
                      Upload File
                    </v-btn>
                    <span class="mr-3">OR</span>
                    <v-btn :disabled="!formFilled || viaUpload" @click="toggleCamera" :class="{ 'success button is-rounded' : !isCameraOpen, 'red button is-rounded' : isCameraOpen}">
                      <span v-if="!isCameraOpen">Open Camera</span>
                      <span v-else>Close Camera</span>
                    </v-btn>
                    <v-btn v-if="isCameraOpen" class="button" color="blue" @click="takePhoto">
                      <span v-if="!isPhotoTaken">Take Photo</span>
                      <span v-else>Retake Photo</span>
                    </v-btn>
                    <v-btn color="success" v-if="isCameraOpen && isPhotoTaken" class="camera-download" @click="downloadImage">
                      <a style="text-decoration:none;color:white;" id="downloadPhoto" v-bind:download="nik + '.jpg'" class="button" role="button"> 
                        Save Photo
                      </a>
                    </v-btn>
                  </div>
                </v-col>
              </v-row>
              <v-row v-if="viaUpload">
                <v-col>
                  <b-form-file
                    v-model="file"
                    :state="Boolean(file)"
                    placeholder="Choose a file or drop it here..."
                    drop-placeholder="Drop file here..."
                  ></b-form-file>
                  <div class="mt-3">Selected file: {{ file ? file.name : '' }}</div>
                </v-col>
              </v-row>
              <v-row>
                <v-col cols="12" md="12">
                  <v-btn
                    block
                    :disabled="!formReadyToSubmit()"
                    color="success"
                    class="mr-0"
                    @click="check">
                    submit
                  </v-btn>
                </v-col>
              </v-row>
            </v-container>
          </v-form>
        </base-material-card>
      </v-col>

    </v-row>
  </v-container>
</template>

<script>
  import axios from 'axios'
  import { BFormFile } from 'bootstrap-vue'
  export default {
    components: {
      'b-form-file': BFormFile,
    },
    data () {
      return {
        nik: '',
        nama: '',
        video: {},
        canvas: {},
        captures: [],
        isCameraOpen: false,
        isPhotoTaken: false,
        imageSaved: false,
        viaUpload: false,
        viaLive: null,
        file: {},
      }
    },
    computed: {
      formFilled () {
        return this.nik !== '' && this.nama !== ''
      },
    },
    mounted () {},
    methods: {
      check (e) {
        e.preventDefault()
        if (this.viaUpload) {
          const formData = new FormData()
          formData.append('file', this.file)
          // You should have a server side REST API
          axios.post('http://localhost:5000/add_employee_upload/' + this.nik + '/' + this.nama, formData, { headers: { 'Content-Type': 'multipart/form-data' } })
            .then(() => {
              console.log('success')
              location.reload()
            })
            .catch(e => {
              console.log(e.toString())
              location.reload()
            })
        } else {
          this.stopCameraStream()
          axios
            .get('http://localhost:5000/add_employee/' + this.nik + '/' + this.nama)
            .then(response => {
              console.log(response)
              alert('Adding Employee Success!')
              location.reload()
            })
        }
      },
      formReadyToSubmit () {
        if (this.viaUpload) {
          // kalau via upload, siap submit berarti semua form telah diisi
          return this.nik !== '' && this.nama !== '' && this.file
        } else {
          // kalau via foto live, siap submit berarti semua form telah diisi dan gambar telah tersimpan di Downloads
          return this.nik !== '' && this.nama !== '' && this.imageSaved && this.isPhotoTaken
        }
      },
      capture () {
        console.log('capture image')
      },
      toggleCamera () {
        if (this.isCameraOpen) {
          this.viaLive = false
          this.isCameraOpen = false
          this.isPhotoTaken = false
          this.stopCameraStream()
        } else {
          this.viaLive = true
          this.isCameraOpen = true
          this.createCameraElement()
        }
      },
      toggleFormUpload () {
        console.log('using upload')
        this.viaUpload = true
      },
      createCameraElement () {
        const constraints = (window.constraints = {
          audio: false,
          video: true,
        })
        navigator.mediaDevices
          .getUserMedia(constraints)
          .then(stream => {
            this.$refs.camera.srcObject = stream
          })
          .catch(error => {
            alert(error)
          })
      },
      stopCameraStream () {
        const tracks = this.$refs.camera.srcObject.getTracks()
        tracks.forEach(track => {
          track.stop()
        })
      },
      takePhoto () {
        // console.log('taking photo')
        this.isPhotoTaken = !this.isPhotoTaken
        const context = this.$refs.canvas.getContext('2d')
        context.drawImage(this.$refs.camera, 0, 0, 450, 337.5)
      },
      downloadImage () {
        if (this.nik === '' || this.nama === '') {
          alert('Please fill the form first before saving photo')
        } else {
          const download = document.getElementById('downloadPhoto')
          const canvas = document.getElementById('photoTaken').toDataURL('image/jpeg')
            .replace('image/jpeg', 'image/octet-stream')
          download.setAttribute('href', canvas)
          // panggil api tiap yang cek status file
          window.setInterval(() => {
            this.isDownloadSuccess(this.nik)
          }, 500)
        }
      },
      isDownloadSuccess (nik) {
        axios
          .get('http://localhost:5000/download_success/' + nik)
          .then(response => {
            // console.log(response.data)
            this.imageSaved = response.data.status === 200
          })
      },
    },
  }
</script>

<style>
</style>
