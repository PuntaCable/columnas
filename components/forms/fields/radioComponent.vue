<template>
  <v-input hide-details>
    <div class="d-flex align-center custom-checkbox fill-width rounded-lg" @click="setClick()"
      :class="{'selected-radio':!active }">
      <v-radio readonly ref="checkbox" outlined class="rounded-lg ma-1" hide-details :value="value">
      </v-radio>&nbsp;<span class="text-h6 font-weight-regular"
        :class="{'primary--text text--darken-1':active}">{{label}}</span>
    </div>
  </v-input>
</template>

<script>
  export default {
    props: {
      readonly:{
        type:Boolean,
        default:false
      },
      value: null,
      label: {
        type: String,
        default: ''
      },
    },
    data(){
      return {
        active: false,
      }
    },
    mounted() {
      this.active = this.$refs.checkbox.isActive
      this.$refs.checkbox.radioGroup.$on('change',(value) => {
        console.log(value)
        this.active = this.value == value
      })
    },
    methods: {
      setClick() {
        if(this.readonly) return
        this.$refs.checkbox.radioGroup.$emit('change', this.value)
      },
    },
  }

</script>

<style scoped lang="scss">
  .custom-checkbox {
    border: 1px solid #9e9e9e;
    height: 56px;
    padding: 0 12px;
  }

  .selected-radio {
    border: 3px solid var(--v-primary-base);
    ;
  }

</style>
