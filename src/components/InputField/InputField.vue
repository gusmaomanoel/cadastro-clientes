<template>
  <div class="input-field">
    <div class="input-field-label">
      <label>{{ label }}</label>
    </div>
    <div class="input-field-input">
      <input
        :class="{
          'form-control': true,
          'is-invalid': required || invalidEmail,
        }"
        :type="type"
        :value="value"
        :maxlength="maxLength"
        @input="onChange"
        @keyup="maskCelPhone($event)"
      />
    </div>
    <div v-if="required" class="required-message">
      <span>Campo obrigatório</span>
    </div>
    <div v-if="!required && invalidEmail" class="required-message">
      <span>Email informado é invalido</span>
    </div>
  </div>
</template>
<script>
export default {
  name: "InputField",
  components: {},
  model: {
    prop: "value",
    event: "input",
  },
  props: {
    label: { type: String, default: null },
    type: { type: String, default: "text" },
    value: { type: String, default: null },
    required: { type: Boolean, default: false },
    invalidEmail: { type: Boolean, default: false },
    maxLength: { type: Number, default: null },
  },
  methods: {
    openClientModal() {
      this.showClientModal = !this.showClientModal;
    },
    onChange(event) {
      this.$emit("input", event.target.value);
    },
    maskCelPhone(event) {
      this.$emit("mask-cel-phone", event);
    },
  },
};
</script>
<style lang="scss">
.input-field {
  display: flex;
  flex-direction: column;
}
.input-field-label {
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
}
.required-message {
  font-size: $mediumBig;
  color: $required;
}
</style>
