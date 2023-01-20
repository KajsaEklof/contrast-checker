<script lang="ts" setup>
import { ref, watch } from "vue";

const contrastData = ref({});
const foregroundColour = ref("");
const backgroundColour = ref("");

watch(foregroundColour, (newValue, oldValue) => {
  console.log(newValue, oldValue);
  checkContrast();
});

watch(backgroundColour, (newValue, oldValue) => {
  console.log(newValue, oldValue);
  checkContrast();
});

function checkContrast() {
  const fColour = foregroundColour.value.replace("#", "");
  const bColour = backgroundColour.value.replace("#", "");

  // Use the WebAIM contrast checker API - https://webaim.org/resources/contrastchecker/
  const url = `https://webaim.org/resources/contrastchecker/?fcolor=${fColour}&bcolor=${bColour}&api`;

  fetch(url)
    .then((response) => response.json())
    .then((data) => (contrastData.value = data))
    .catch((error) => {
      console.log("error", error);
    });
}
</script>

<template>
  <div>
    <h1>Contrast Checker</h1>
    <div>
      <label for="foreground">Foreground Colour</label>
      <input
        v-model="foregroundColour"
        type="color"
        name="foreground"
        id="foreground"
      />
      <label for="background">Background Colour</label>
      <input
        v-model="backgroundColour"
        type="color"
        name="background"
        id="background"
      />
    </div>

    <pre>{{ contrastData }}</pre>
  </div>
</template>
