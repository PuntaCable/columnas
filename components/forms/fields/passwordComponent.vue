<template>
  <div class="d-flex flex-column">
    <label class="font-weight-regular mb-2 text-uppercase text-subtitle-2" :class="labelColor">{{label}}</label>
    <v-text-field prepend-inner-icon="mdi-lock" outlined class="rounded-lg font-weight-regular" :type="typeInput" :append-icon="appendIcon" @click:append="show = !show"
      solo ref="input" hide-details v-model="fieldValue" v-bind="props">
    </v-text-field>
  </div>
</template>

<script>
  export default {
    inheritAttrs: false,
    props: {
      value: null,
      "label-color": {
        type: String,
        default: "grey--text text--darken-4"
      },
      label: {
        type: String,
        default: ''
      },
    },
    data() {
      return {
        show: false,
        focused: false,
        fieldValue: this.value,
      }
    },
    methods: {},
    watch: {
      fieldValue(newValue) {
        this.$emit('input', newValue);
      },
      value(value) {
        this.fieldValue = value
      },
      "$refs": {
        deep: true,
        handler(newValue) {
          console.log(console.log('isValid', this.$refs.input.validate()))
        }
      },
    },
    computed: {
      appendIcon() {
        return this.show ? 'mdi-eye' : 'mdi-eye-off'
      },
      typeInput() {
        return this.show ? 'text' : 'password'
      },
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

<style>

</style>
