<template>
  <div class="file-manager-container">
    <div
      class="file-manager-actions"
    >
      <q-btn
        dense
        label="ATT"
        @click="listarArquivos('', 'arquivos')"
      />
      <q-btn
        dense
        label="Apagar Tudo"
        @click="cleanAll"
      />
      <q-btn
        dense
        label="Apagar"
        @click="apagar"
      />
      <q-btn
        dense
        label="Apagar Arquivo"
        @click="apagarArquivo"
      />
      <q-btn
        dense
        label="Nova Pasta"
        @click="novaPasta"
      />
      <q-btn
        dense
        label="Novo Arquivo txt"
        @click="novaArquivo"
      />
      <q-btn
        dense
        label="Space"
        @click="getStorangeSpace"
      />
      <div class="store">
        <div>
          <q-icon name="storage" />
          <span>{{ storange.used }} - {{ storange.total }} - ({{ (storange.procent).toFixed(0) }}%)</span>
        </div>
        <div class="storange">
          <div class="storange-space" :style="{ width: storange.procent + '%' }"/>
        </div>
      </div>

    </div>
    <div class="file-manager-files">
      <div class="folders">
        <ul class="folder-list">
          <li
            :class="{'selected': selectedFolder === item.label }"
            v-for="(item, index) in getFolders"
            :key="index"
            @click="selectFolder(item)"
          >
            <q-icon name="folder" />
            <span>{{ item.label }}</span>
          </li>
        </ul>
      </div>
      <div class="files">
        <ul class="files-list">
          <li
            :class="{'selected': selected === index }"
            v-for="(file, index) in folderFiles"
            :key="index"
            @dblclick="baixar(file.label)"
          >
            <q-icon name="folder" />
            <span>{{ file.label }}</span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import ChromeStore from './chromestore'
import swal from 'sweetalert2'
import { url } from './image'
export default {
  name: 'FileManager',
  created () {
    this.inicialize()
      .then(() => {
        this.listarArquivos('', 'arquivos')
      })
  },
  mounted () {
  },
  data: () => ({
    pastas: [],
    arquivos: [],
    folderFiles: [],
    selected: null,
    selectedFolder: '',
    pathBase: '/',
    storange: {
      used: 0,
      total: 1024 * (1024 * 10),
      procent: 0
    },
    chromeStore: undefined
  }),
  methods: {
    inicialize () {
      return new Promise((resolve, reject) => {
        this.chromeStore = new ChromeStore([{ path: '/' }])
        this.chromeStore.init(this.storange.total, function (cstore) {
          return resolve()
        })
      })
    },
    getStorangeSpace () {
      const $this = this
      this.chromeStore.usedAndRemaining(function (used, remaining) {
        $this.storange.used = used
        $this.storange.procent = ((used * 100) / $this.storange.total)
      })
    },
    selectFolder (item) {
      this.selectedFolder = item.label
      this.listarArquivos(item.label, 'folderFiles')
    },
    novaPasta () {
      swal.fire({
        title: '',
        text: 'Nome',
        input: 'text',
        inputAttributes: {
          autocapitalize: 'off'
        },
        showCancelButton: true,
        showCloseButton: true,
        customClass: '',
        preConfirm: (nome) => {
          return this.criarPasta(nome)
        }
      })
    },
    novaArquivo () {
      swal.fire({
        title: '',
        text: 'Nome',
        input: 'text',
        inputAttributes: {
          autocapitalize: 'off'
        },
        showCancelButton: true,
        showCloseButton: true,
        customClass: '',
        preConfirm: (nome) => {
          return this.criarArquivo(nome)
        }
      })
    },
    listarArquivos (pasta, destination) {
      const $this = this
      $this.$data[destination] = []
      this.folderFiles = []
      this.chromeStore.getDir((`${this.pathBase}/${pasta}`).replace(/\/\//g, '/'), { create: false }, function (dirEntry) {
        console.log('~> ', dirEntry)
        $this.chromeStore.ls(pasta, function (arr) {
          $this.$data[destination] = arr.map((item) => ({ label: item.name }))
        })
      })
      $this.getStorangeSpace()
    },
    criarPasta (nome) {
      let path = this.pathBase
      if (this.selectedFolder) {
        path = `${this.pathBase}/${this.selectedFolder}`
      }
      const $this = this
      // eslint-disable-next-line no-unused-vars
      this.chromeStore.getDir((`${path}/${nome}`).replace(/\/\//g, '/'), { create: true }, function (dirEntry) {
        $this.listarArquivos('', 'arquivos')
      })
    },
    criarArquivo (nome) {
      let path = this.pathBase
      const $this = this
      // Create Directory
      if ($this.selectedFolder) {
        path = `${path}/${this.selectedFolder}`
      }
      this.chromeStore.getDir((`${path}`).replace(/\/\//g, '/'), { create: true }, function () {
        // Create and write to file
        $this.chromeStore.write((`${path}/${nome}.txt`).replace(/\/\//g, '/'), 'text/plain', url, { create: true }, function () {
          $this.listarArquivos('', 'arquivos')
        })
      })
    },
    apagar () {
      const $this = this
      let path = this.pathBase
      if (this.selectedFolder) {
        path = `${this.pathBase}/${this.selectedFolder}`
      }
      this.chromeStore.deleteDir(path, { recursive: true }, function () {
        $this.listarArquivos('', 'arquivos')
        $this.selectedFolder = ''
      })
    },
    apagarArquivo () {
      this.chromeStore.deleteFile(this.selectedFolder)
    },
    cleanAll () {
      this.selectedFolder = ''
      const $this = this
      this.chromeStore.ls((this.pathBase).replace(/\/\//g, '/'), function (arr) {
        var length = arr.length

        for (var i = 0; i < length; ++i) {
          $this.chromeStore.deleteDir(`/${arr[i].name}`, { recursive: true }, function () {
            $this.listarArquivos('', 'arquivos')
          })
        }
      })
    },
    baixar (nome) {
      let path = this.pathBase
      if (this.selectedFolder) {
        path = `${this.pathBase}/${this.selectedFolder}`
      }
      this.chromeStore.getFile((`${path}/${nome}`).replace(/\/\//g, '/'), { create: false, exclusive: true }, function (fileEntry) {
        fileEntry.file(function (file) {
          const reader = new FileReader()
          reader.onload = (e) => {
            console.log('~> ', e.target.result)
            const link = document.createElement('a')
            link.download = `${file.name}`
            link.href = reader.result
            link.click()
          }
          reader.readAsDataURL(file)
        })
      })
    }
  },
  computed: {
    getFolders () {
      // return this.arquivos.filter((item) => item.label.indexOf('.') === -1)
      return this.arquivos
    }
  }
}
</script>

<style
  lang="stylus"
  rel="stylesheet/stylus"
  scoped
>
.file-manager-container
  width: 96%;
  height: calc(100vh - 116px);
  border 1px solid #000
  padding 0 0 10px 0
  .file-manager-actions
    padding 5px 20px
    border: 1px solid #ddd;
    width: 100%;
    display: flex;
    flex-direction: row;
    position relative
    .q-btn
      margin 0 5px
    .store
      position: absolute;
      right: 0;
      top: 0;
      height: 100%;
      width: 230px;
      padding: 0 5px 0 0;
      .storange
        width: 100%;
        border: 1px solid;
        height: 15px;
        .storange-space
          height: 100%;
          background: brown;
  .file-manager-files
    display flex
    flex-direction row;
    height calc(100% - 43px)
    width 100%
    .files
      width calc(100% - 300px)
      border-style solid
      border-width 1px 0 0 0
      border-color #666666
      .files-list
        list-style: none;
        padding: 0;
        overflow-y: auto;
        overflow-x: hidden;
        max-height: 100%;
        margin: 0;
        li
          padding: 10px;
          .q-icon
            font-size 17px
            margin 0 10px
          &:hover
            background #6666662e
    .folders
      width 300px
      border-style solid
      border-width 1px 1px 0 0
      border-color #666666
      .folder-list
        list-style: none;
        padding: 0;
        overflow-y: auto;
        overflow-x: hidden;
        max-height: 100%;
        margin: 0;
        li
          padding: 10px;
          .q-icon
            font-size 17px
            margin 0 10px
          &:hover
            background #6666662e
.selected
  background #6666662e
</style>
