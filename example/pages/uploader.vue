<template>
  <div class="page">
    <wv-uploader :disabled="true"/>
  </div>
</template>

<script>
import thumb from '../assets/images/wevue_placeholder.png'

export default {
  data () {
    return {
      thumb
    }
  }
}
</script>
