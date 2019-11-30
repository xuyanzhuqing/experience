<template>
  <div>
    3
    <p>where come from ?</p>
    <p>{{oldView}}</p>

    <Input v-model="modal.title"/>
  </div>
</template>
<script>
export default {
  name: 'step3',
  inject: ['modal', 'jump', 'safeClose'],
  props: {
    oldView: {
      required: false,
      type: String,
      default: ''
    }
  },
  methods: {
    qaPreview () {
      this.jump(this.oldView)
    },
    qaSubmit () {
      alert('hoo, i get it')
      this.safeClose()
    }
  }
}
</script>