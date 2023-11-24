<script setup>
import HelloWorld from './components/HelloWorld.vue'
import ColumnVue from './components/ColumnVue.vue'

import { ref } from 'vue';

// Reactive properties
const selected = ref(null);

// Methods
const handleSelection = (item) => {
  selected.value = item;
  // Additional logic
};

const equipmentData = [
  {
    "uuid": "44ad25ed-6a4c-489e-beb9-3ffb9a1229be",
    "name": "Main Control Unit",
    "description": "",
    "type": "Controller",
    "date_time_created": "2023-11-22T22:23:52.826779",
    "date_time_updated": "2023-11-22T22:23:52.826784",
    "user_defined_properties": {
      "manufacturer": "TechCorp",
      "model": "XG-5000"
    },
    "children": [
      {
        "uuid": "310e6903-0672-4809-aaee-682d79efc32b",
        "name": "Sub Control Unit 1 with a very long description",
        "description": "",
        "type": "Controller",
        "date_time_created": "2023-11-22T22:23:52.827138",
        "date_time_updated": "2023-11-22T22:23:52.827140",
        "user_defined_properties": {
          "manufacturer": "TechCorp",
          "series": "SC-1000"
        },
        "children": [
          {
            "uuid": "249f6ea3-0f4f-4697-af6b-8c19e1f38752",
            "name": "Sensor Module 1",
            "description": "",
            "type": "Sensor",
            "date_time_created": "2023-11-22T22:23:52.827208",
            "date_time_updated": "2023-11-22T22:23:52.827210",
            "user_defined_properties": {
              "sensor_type": "Temperature",
              "precision": "0.5°C"
            },
            "children": [
              {
                "uuid": "d3579e57-7610-40de-ae2f-23404a3fe2a7",
                "name": "Micro Sensor 1",
                "description": "",
                "type": "Sensor",
                "date_time_created": "2023-11-22T22:23:52.827294",
                "date_time_updated": "2023-11-22T22:23:52.827295",
                "user_defined_properties": {
                  "resolution": "100dpi",
                  "connectivity": "Wireless"
                },
                "children": []
              },
              {
                "uuid": "2c5b3f19-8921-43ea-af96-6dc3e457ff19",
                "name": "Micro Sensor 2",
                "description": "",
                "type": "Sensor",
                "date_time_created": "2023-11-22T22:23:52.827759",
                "date_time_updated": "2023-11-22T22:23:52.827764",
                "user_defined_properties": {
                  "resolution": "110dpi",
                  "connectivity": "Wireless"
                },
                "children": []
              }
            ]
          }
        ]
      },
      {
        "uuid": "sub-control-unit-2-uuid",
        "name": "Sub Control Unit 2",
        "description": "",
        "type": "Controller",
        "date_time_created": "2023-11-22T22:23:52.827442",
        "date_time_updated": "2023-11-22T22:23:52.827442",
        "user_defined_properties": {
          "manufacturer": "TechCorp",
          "series": "SC-2000"
        },
        "children": [
          {
            "uuid": "sensor-module-3-uuid",
            "name": "Sensor Module 3",
            "description": "",
            "type": "Sensor",
            "date_time_created": "2023-11-22T22:23:52.827442",
            "date_time_updated": "2023-11-22T22:23:52.827442",
            "user_defined_properties": {
              "sensor_type": "Pressure",
              "range": "30-60 PSI"
            },
            "children": [
              {
                "uuid": "pressure-sensor-1-uuid",
                "name": "Pressure Sensor 1",
                "description": "",
                "type": "Sensor",
                "date_time_created": "2023-11-22T22:27:19.295966",
                "date_time_updated": "2023-11-22T22:27:19.295967",
                "user_defined_properties": {
                  "sensitivity": "0.1 PSI",
                  "connectivity": "Wired"
                },
                "children": []
              },
              {
                "uuid": "pressure-sensor-2-uuid",
                "name": "Pressure Sensor 2",
                "description": "",
                "type": "Sensor",
                "date_time_created": "2023-11-22T22:27:19.295979",
                "date_time_updated": "2023-11-22T22:27:19.295980",
                "user_defined_properties": {
                  "sensitivity": "1.1 PSI",
                  "connectivity": "Wired"
                },
                "children": []
              }
            ]
          }
        ]
      }
    ]
  }
];

</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />
    </div>
  </header>

  <main>
    <column-vue :equipments="equipmentData" @select="handleSelection"></column-vue>

  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
