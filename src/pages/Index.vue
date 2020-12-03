<template>
  <q-page class="flex flex-center">
    <file-manager />
  </q-page>
</template>

<script>
import FileManager from 'components/FileManager'
export default {
  name: 'PageIndex',
  components: { FileManager }
}
</script>
