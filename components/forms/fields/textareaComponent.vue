<template>
  <div class="d-flex flex-column">
    <label class="font-weight-regular mb-2 text-uppercase text-subtitle-2" :class="labelColor">{{label}}</label>
    <v-textarea class="rounded-lg font-weight-regular" solo ref="input"  hide-details v-model="fieldValue" @focus="checkFocus()" v-bind="props">
      <template v-slot:append="{item}">
        <span></span>
        <template v-if="focused">
          <v-icon v-if="!isValid" class="yellow">mdi-alert-circle</v-icon>
          <v-icon v-else color="success">{{item}}</v-icon>
        </template>
      </template>
    </v-textarea>
  </div>
</template>

<script>
  export default {
    inheritAttrs: false,
    props: {
      value: null,
      "label-color":{
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
        focused:false,
        fieldValue:this.value,
      }
    },
    updated() {
    },
    methods: {
      checkFocus() {
        if(!this.focused) {
          this.focused = true
        }
        this.$forceUpdate()
      }
    },
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
      isValid(){
        if(this.$refs.input) {
          return this.$refs.input.validate()
        }
      },
      props() {
        const props = JSON.parse(JSON.stringify(this.$props))
        delete props.value
        delete props.label
        return {...props,...this.$attrs}
      }
    }
  }

</script>

<style>

</style>
