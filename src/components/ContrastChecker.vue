<script lang="ts" setup>
import { onBeforeMount, reactive, ref, watch } from "vue";
import { Icon } from "@iconify/vue";
import ThemeColours from "./ThemeColours.vue";

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

function checkContrast() {
  const fColour = foregroundColour.value.replace("#", "");
  const bColour = backgroundColour.value.replace("#", "");

  // Use the WebAIM contrast checker API - https://webaim.org/resources/contrastchecker/
  const url = `https://webaim.org/resources/contrastchecker/?fcolor=${fColour}&bcolor=${bColour}&api`;

  fetch(url)
    .then((response) => response.json())
    .then((data) => {
      console.log("data", data);
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
    <div class="d-flex align-center justify-between colour-checker">
      <div class="d-flex column ratio-levels">
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
              {{ result }}
              <Icon :icon="icons[result]" />
            </span>
            <span class="level"> {{ level }} </span>
          </div>
        </div>
      </div>
      <div class="colour-pickers d-flex">
        <fieldset>
          <legend>Foreground</legend>
          <div class="d-flex align-center">
            <label hidden for="foreground-colour-input">Foreground</label>
            <input
              v-model="foregroundColour"
              type="color"
              id="foreground-colour-input"
            />
            <span>{{ foregroundColour }}</span>
          </div>
        </fieldset>
        <fieldset>
          <legend>Background</legend>
          <div class="d-flex align-center">
            <label hidden for="readonly-background"></label>
            <input
              v-model="backgroundColour"
              type="color"
              id="readonly-background"
            />
            <span>{{ backgroundColour }}</span>
          </div>
        </fieldset>
      </div>
    </div>
    <ThemeColours
      :background="backgroundColour"
      :foreground="foregroundColour"
    />
  </div>
</template>
