<script lang="ts" setup>
import { onBeforeMount, reactive, ref, watch } from "vue";
import { Icon } from "@iconify/vue";
import ThemeColours from "./ThemeColours.vue";
import { ColorPicker } from "vue-accessible-color-picker";

const contrastData: { [key: string]: string } = reactive({});
const ratio = ref("");
const foregroundColour = ref("#000000");
const backgroundColour = ref("#ffffff");
const icons: { [key: string]: string } = {
  pass: "material-symbols:check-small-rounded",
  fail: "material-symbols:close-rounded",
};

watch([foregroundColour, backgroundColour], () => {
  console.log(foregroundColour, backgroundColour);
  checkContrast();
});

onBeforeMount(() => {
  checkContrast();
});

function updateColor(
  eventData: { [key: string]: { [key: string]: string } | string },
  type: string
): void {
  console.log(eventData, type);
  console.log("type", type);
  switch (type) {
    case "foreground":
      foregroundColour.value = eventData.cssColor as string;
      break;
    case "background":
      backgroundColour.value = eventData.cssColor as string;
  }
}

function checkContrast() {
  const fColour = foregroundColour.value.replace("#", "");
  const bColour = backgroundColour.value.replace("#", "");

  // Use the WebAIM contrast checker API - https://webaim.org/resources/contrastchecker/
  const url = `https://webaim.org/resources/contrastchecker/?fcolor=${fColour}&bcolor=${bColour}&api`;

  fetch(url)
    .then((response) => response.json())
    .then((data) => {
      console.log(data);
      ratio.value = data.ratio;
      contrastData.AA = data.AA;
      contrastData.AALarge = data.AALarge;
      contrastData.AAA = data.AAA;
      contrastData.AAALarge = data.AAALarge;
    })
    .catch((error) => {
      // console.log("error", error);
    });
}
</script>

<template>
  <div
    :style="{ backgroundColor: backgroundColour, color: foregroundColour }"
    class="contrast-checker"
  >
    <h1>Colour Contrast Checker</h1>
    <div class="d-flex align-start justify-between">
      <div class="d-flex column">
        <div class="ratio-wrapper">
          <span>Aa</span><span>Ratio: {{ ratio }}</span>
        </div>
        <div class="levels">
          <div
            v-for="(result, level) in contrastData"
            :key="level"
            class="level-wrapper justify-center"
          >
            <span
              :class="[
                'grade d-flex align-center',
                { fail: result === 'fail', pass: result === 'pass' },
              ]"
            >
              <Icon :icon="icons[result]" />
              {{ result }}
            </span>
            <span class="level"> {{ level }} </span>
          </div>
        </div>
      </div>
      <div class="colour-pickers d-flex">
        <div class="d-flex column">
          <h4>Foreground</h4>
          <ColorPicker
            :color="foregroundColour"
            :visible-formats="['hex']"
            default-format="hex"
            class="foreground"
            @color-change="updateColor($event, 'foreground')"
          />
        </div>
        <div class="d-flex column">
          <h4>Background</h4>
          <ColorPicker
            :color="backgroundColour"
            :visible-formats="['hex']"
            default-format="hex"
            class="background"
            @color-change="updateColor($event, 'background')"
          />
        </div>
      </div>
    </div>
    <!-- <ThemeColours /> -->
    <!-- <pre>{{ contrastData }}</pre> -->
  </div>
</template>
