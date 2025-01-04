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
    <v-row>
      <button style="font-size: 100" @click="toggleElectricField">⚡️</button>
      <p style="width: 200px; margin: 20 auto;">
        Prędkość dryfu: {{ driftVelocity }}
      </p>
    </v-row>
    <v-row justify="center" style="margin: 20px">
      <PhysicsModel
        :temperature="temperature"
        :carrierConcentration="carrierConcentration"
        :atomSize="atomSize"
        :isElectricFieldOn="isElectricFieldOn"
        @drift-velocity="driftVelocity = $event"
      />
  </v-row>
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
        driftVelocity: 0,
        temperature: 0,
        carrierConcentration: 1,
        atomSize: 30,
        temperatureMin: -273,
        temperatureMax: 2000,
        carrierConcentrationMin: 1,
        carrierConcentrationMax: 100,
        atomSizeMin: 10,
        atomSizeMax: 40,
        isElectricFieldOn: true,
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
      toggleElectricField() {
        this.isElectricFieldOn = !this.isElectricFieldOn;
        console.log(this.isElectricFieldOn)
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
