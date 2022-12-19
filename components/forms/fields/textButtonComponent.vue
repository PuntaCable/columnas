<template>
  <div class="fill-width">
    <v-input v-bind="$attrs['class']" class="fill-width d-flex flex-column fill-width" hide-details>
      <v-text-field outlined class="elevation-2 rounded-lg rounded-r-0 font-weight-regular input-width" ref="input" hide-details v-model="fieldValue" v-bind="$attrs"
        @focus="checkFocus()">
      </v-text-field>
      <v-btn class="button-width rounded-l-0 black--text rounded-lg font-weight-regular" :color="buttonColor" height="55" x-large @click="handler()">
        {{$data["button-label"]}}&nbsp;<v-icon color="white">{{icon}}</v-icon>
      </v-btn>
    </v-input>
    <v-snackbar color="success" v-model="actionSuccess">
      {{notificationText}}
    </v-snackbar>
  </div>
</template>

<script>
  export default {
    inheritAttrs: false,
    props: {
      "notification-text": {
        type: String,
        default: '',
      },
      click: {
        type: Function,
        default: () => {}
      },
      value: null,
      "button-label":{
        type: String,
        default: 'Guardar'
      },
      icon: {
        type: String,
        default: 'mdi-content-save'
      },
      buttonColor: {
        type: String,
        default: 'primary'
      }
    },
    data() {
      return {
        fieldValue: this.value,
        isValid: null,
        focused: false,
        actionSuccess: false
      }
    },
    updated() {
      this.isValid = this.$refs.input.validate()
    },
    methods: {
      handler() {
        this.$emit('click',this.fieldValue)
        this.actionSuccess = true
      },
      checkFocus() {
        if (!this.focused) {
          this.focused = true
        }
        this.$forceUpdate()
      }
    },
    watch: {
      fieldValue(newValue) {
        this.$emit('input', newValue);
      }
    },
    computed: {
      props() {
        const props = JSON.parse(JSON.stringify(this.$props))
        delete props.value
        delete props.label
        return {
          ...props,
          ...this.$attrs
        }
      }
    }
  }

</script>

<style scoped lang="scss">
.input-width {
  width: 90%;
}
.button-width{
    width: 10%;
}
</style>
