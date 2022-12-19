<template>
<div>
  <v-chip label filter-icon="mdi-check-circle"  v-bind="$attrs" @click:close="clickClose()" :class="returnClass" filter class="font-weight-regular mr-2">
    <slot></slot>
  </v-chip>
  <v-snackbar color="warning"  v-model="notificationDelete">
    {{notificationText}}
  </v-snackbar>
</div>
</template>

<script>
  export default {
    props: {
      "notification-text": {
        type: String,
        default: '',
      },
    },
    data() {
      return {
      notificationDelete: false
      }
    },
    methods: {
      clickClose() {
        this.$emit('click:close')
        this.notificationDelete = true

      },
      returnClass() {
        if (this.selected) {
          return 'primary'
        } else {
          return 'grey lighten-2'
        }
      }
    }
  }

</script>

<style>

</style>
