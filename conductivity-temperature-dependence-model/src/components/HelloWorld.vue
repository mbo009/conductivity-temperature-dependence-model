<template>
  <v-container>
    <v-row align="center" justify="space-between">
      <SliderWithField
        v-model="temperature"
        :min="temperatureMin"
        :max="temperatureMax"
        label="Temperatura"
      />

      <SliderWithField
        v-model="carrierConcentration"
        :min="carrierConcentrationMin"
        :max="carrierConcentrationMax"
        label="Koncentracja nośników"
      />

      <SliderWithField
        v-model="atomSize"
        :min="atomSizeMin"
        :max="atomSizeMax"
        label="Rozmiar atomów"
      />
    </v-row>
    <p style="text-align: center">Prędkość dryfu:</p>
    <PhysicsModel />
  </v-container>
</template>

<script>
import SliderWithField from "@/components/SliderWithField.vue";
export default {
  components: {
    SliderWithField,
  },
  data() {
    return {
      temperature: 0,
      carrierConcentration: 1,
      atomSize: 1,
      temperatureMin: -273,
      temperatureMax: 2000,
      carrierConcentrationMin: 1,
      carrierConcentrationMax: 100,
      atomSizeMin: 1,
      atomSizeMax: 100,
    };
  },
  methods: {
    validateInput(variable, min, max) {
      if (this[variable] < min) {
        this[variable] = min;
      } else if (this[variable] > max) {
        this[variable] = max;
      }
    },
  },
};
</script>
