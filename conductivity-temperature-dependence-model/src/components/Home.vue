<template>
  <v-container>
    <v-row justify="space-evenly">
      <SliderWithField
        v-model="temperature"
        :min="temperatureMin"
        :max="temperatureMax"
        label="Temperatura [℃]"
      />

      <SliderWithField
        v-model="atomMass"
        :min="atomMassMin"
        :max="atomMassMax"
        label="Masa atomu [u]"
      />

      <SliderWithField
        v-model="atomSize"
        :min="atomSizeMin"
        :max="atomSizeMax"
        label="Rozmiar atomu"
      />
    </v-row>
    
    <v-row justify="space-evenly">
      <SliderWithField
        v-model="carrierConcentration"
        :min="carrierConcentrationMin"
        :max="carrierConcentrationMax"
        label="Koncentracja nośników"
      />
      
      <SliderWithField
        v-model="electricFieldForce"
        :min="electricFieldForceMin"
        :max="electricFieldForceMax"
        label="Pole elektryczne [N/C]"
      />

      <SliderWithField
        v-model="carrierSize"
        :min="carrierSizeMin"
        :max="carrierSizeMax"
        label="Rozmiar nośnika"
      />
    </v-row>

    <v-row justify="center" style="margin: 40px 0 20px 0;">
      <PhysicsModel
        :temperature="temperature"
        :carrierConcentration="carrierConcentration"
        :atomSize="atomSize"
        :atomMass="atomMass"
        :carrierSize="carrierSize"
        :electricFieldForce="electricFieldForce"
        :isElectricFieldOn="isElectricFieldOn"
        :isInteractive="isInteractive"
        @drift-velocity="driftVelocity = $event"
      />
    </v-row>
    
    <v-row align="center" justify="center">
      <p style="width: 250px; margin: 20 auto; font-size: 20px">
        Prędkość dryfu: {{ driftVelocity }}
      </p>
    </v-row>

    <v-row justify="space-evenly">
      <v-btn @click="toggleElectricField" :color="isElectricFieldOn ? 'green' : 'red'">
        <span>Pole elektryczne</span>
      </v-btn>

      <v-btn @click="toggleInteractiveMode" :color="isInteractive ? 'green' : 'red'">
        <span>Tryb interaktywny</span >
      </v-btn>
    </v-row>
  </v-container>
</template>


<script>
export default {
  data() {
    return {
      driftVelocity: 0,

      temperature: 0,
      temperatureMin: -273,
      temperatureMax: 2000,
      
      carrierConcentration: 1,
      carrierConcentrationMin: 1,
      carrierConcentrationMax: 10,

      atomSize: 30,
      atomSizeMin: 10,
      atomSizeMax: 40,

      atomMass: 6,
      atomMassMin: 6, // Lit
      atomMassMax: 190, // Osm

      carrierSize: 5,
      carrierSizeMin: 1,
      carrierSizeMax: 10,

      electricFieldForce: 3,
      electricFieldForceMin: 1,
      electricFieldForceMax: 50,

      isElectricFieldOn: true,
      isInteractive: false,
    };
  },
  methods: {
    toggleElectricField() {
      this.isElectricFieldOn = !this.isElectricFieldOn;
    },
    toggleInteractiveMode() {
      this.isInteractive = !this.isInteractive;
    }
  },
};
</script>
