<template>
  <v-col :xs="12" :sm="4">
    <v-row align="center" justify="space-between">
      <v-slider
        v-model="internalModelValue"
        :min="min"
        :max="max"
        step="1"
        :label="label"
        class="mt-5"
      ></v-slider>
      <v-text-field
        v-model="internalModelValue"
        :min="min"
        :max="max"
        class="mt-5"
        type="number"
        @blur="validate"
        hide-details
        style="max-width: 100px"
      ></v-text-field>
    </v-row>
  </v-col>
</template>

<script>
export default {
  props: {
    modelValue: {
      type: Number,
      required: true,
    },
    min: {
      type: Number,
      required: true,
    },
    max: {
      type: Number,
      required: true,
    },
    label: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      internalModelValue: this.modelValue,
    };
  },
  watch: {
    modelValue(newVal) {
      this.internalModelValue = newVal;
    },
    internalModelValue(newVal) {
      this.$emit("update:modelValue", newVal);
    },
  },
  methods: {
    validate() {
      if (this.internalModelValue < this.min) {
        this.internalModelValue = this.min;
      } else if (this.internalModelValue > this.max) {
        this.internalModelValue = this.max;
      }
    },
  },
};
</script>
