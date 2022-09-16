<template>
  <div class="control is-3">
    <input
      v-model="displayValue"
      class="input"
      type="text"
      @blur="isInputActive = false"
      @focus="isInputActive = true"
    />
  </div>
</template>
<script>
export default {
  data() {
    return {
      isInputActive: false
    }
  },
  props: ['value'],

  computed: {
    displayValue: {
      // eslint-disable-next-line
      get() {
        if (this.isInputActive) {
          // Cursor is inside the input field. unformat display value for user
          return this.value.toString()
        } else {
          // User is not modifying now. Format display value for user interface
          return (
            '$ ' +
            this.value.toFixed(2).replace(/(\d)(?=(\d{3})+(?:\.\d+)?$)/g, '$1,')
          )
        }
      },
      // eslint-disable-next-line
      set(modifiedValue) {
        // Recalculate value after ignoring "$" and "," in user input
        // eslint-disable-next-line
        let newValue = parseFloat(modifiedValue.replace(/[^\d\.]/g, ''))
        // Ensure that it is not NaN
        if (isNaN(newValue)) {
          newValue = 0
        }
        // Note: we cannot set this.value as it is a "prop". It needs to be passed to parent component
        // $emit the event so that parent component gets it
        this.$emit('input', newValue)
      }
    }
  }
}
</script>
