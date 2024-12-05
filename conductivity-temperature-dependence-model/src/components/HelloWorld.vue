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

      <v-select
        :items="materialNames"
        v-model="selectedMaterial"
        label="Wybierz materiał"
        @change="updateTemperature"
      ></v-select>
    </v-row>
    <p style="text-align: center">Prędkość dryfu:</p>
    <PhysicsModel
      :temperature="temperature"
      :carrierConcentration="carrierConcentration"
      :atomSize="atomSize"
      :debeyeTemperature="debeyeTemperature"
    />
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
      atomSize: 30,
      temperatureMin: -273,
      temperatureMax: 2000,
      carrierConcentrationMin: 1,
      carrierConcentrationMax: 100,
      atomSizeMin: 10,
      atomSizeMax: 40,

      materials: new Map([
        ["Aluminium", 426],
        ["Platyna", 240],
        ["Kadm", 186],
        ["Chrom", 610],
        ["Srebro", 225],
        ["Miedź", 344.5],
        ["Złoto", 165],
        ["Tytan", 420],
        ["Żelazo", 464],
        ["Wolfram", 405],
        ["Nikiel", 440],
        ["Cynk", 300],
      ]),

      selectedMaterial: "Aluminium",
    };
  },

  computed: {
    materialNames() {
      return Array.from(this.materials.keys());
    },
    debeyeTemperature() {
      return this.materials[this.selectedMaterial];
    },
  },

  methods: {
    validateInput(variable, min, max) {
      if (this[variable] < min) {
        this[variable] = min;
      } else if (this[variable] > max) {
        this[variable] = max;
      }
    },

    updateTemperature(selectedMaterial) {
      const temperature = this.materials.get(selectedMaterial);
      if (temperature !== undefined) {
        this.temperature = temperature;
      }
    },
  },
};
</script>
