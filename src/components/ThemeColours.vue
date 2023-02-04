<script lang="ts" setup>
import { ref, toRefs, watch } from "vue";
import { Icon } from "@iconify/vue";

interface ContrastResult {
  [key: string]: string;
}

const props = defineProps({
  background: {
    type: String,
    required: true,
  },
  foreground: {
    type: String,
    required: true,
  },
});

const { background, foreground } = toRefs(props);
const isLoadingThemeResult = ref(false);
const themeColoursInput = ref("");
const themeColours: { [key: string]: any } = ref({});
const icons: { [key: string]: string } = {
  pass: "material-symbols:check-small-rounded",
  fail: "material-symbols:close-rounded",
};

watch(background, () => {
  setThemeColours();
});

async function setThemeColours(): Promise<{ [key: string]: any }> {
  if (themeColoursInput.value === "") {
    return {};
  }

  const data: { [key: string]: any } = {};
  const jsonData = JSON.parse(themeColoursInput.value);
  const keys = Object.keys(jsonData);
  isLoadingThemeResult.value = true;

  await Promise.all(
    keys.map(async (colour: string) => {
      data[colour] = {
        name: colour,
        colour: jsonData[colour],
      };

      if (!background?.value) {
        return;
      }

      const response = await checkContrast(
        jsonData[colour].replace("#", ""),
        background.value.replace("#", "")
      );

      if (!response) {
        return data;
      }

      const ratio = response.ratio;
      const levels = Object.keys(response)
        .filter((x) => x !== "ratio")
        .map((key: string) => {
          return {
            level: key,
            result: response[key] as string,
          };
        });

      data[colour] = { ...data[colour], ratio, levels };
    })
  );

  themeColours.value = data;

  setTimeout(() => {
    isLoadingThemeResult.value = false;
  }, 200);

  return data;
}

async function checkContrast(
  colour1: string,
  colour2: string
): Promise<ContrastResult | undefined> {
  // Use the WebAIM contrast checker API - https://webaim.org/resources/contrastchecker/
  const url = `https://webaim.org/resources/contrastchecker/?fcolor=${colour1}&bcolor=${colour2}&api`;
  let response = undefined;

  await fetch(url)
    .then((response) => response.json())
    .then((data) => {
      console.log(data);
      response = data;

      return data;
    })
    .catch((error) => {
      console.log("error", error);
    });

  return response;
}

function autoGrow() {
  const textareaEl = document.getElementById("themeColours");
  if (!textareaEl) {
    return;
  }

  if (textareaEl.scrollHeight > textareaEl.clientHeight) {
    textareaEl.style.height = `${textareaEl.scrollHeight}px`;
  }
}
</script>
<template>
  <div class="d-flex column theme-checker">
    <h2>Theme Checker</h2>
    <p>
      Paste a json colour theme below to check the colour contrast for each
      colour against the current background colour.
    </p>
    <div class="d-flex">
      <div class="theme-colours">
        <label for="themeColours">Theme colours:</label>
        <textarea
          v-model="themeColoursInput"
          id="themeColours"
          name="themeColours"
          rows="10"
          @keyup="autoGrow"
          @input="setThemeColours"
        ></textarea>
      </div>
      <div class="results">
        <Transition name="fade" mode="out-in">
          <div v-if="isLoadingThemeResult" class="d-flex align-center">
            <span :style="{ color: foreground }" class="loader"></span>
            <span>Calculating...</span>
          </div>
        </Transition>
        <template v-if="!isLoadingThemeResult">
          <div
            v-for="(data, colour) in themeColours"
            :key="colour"
            :style="{ color: data.colour }"
            class="d-flex column result-wrapper"
          >
            <div class="result">
              <span class="theme-name">{{ colour }}:</span>
            </div>
            <div class="levels">
              <div
                v-for="level in data.levels"
                :key="level"
                class="level-wrapper"
              >
                <span
                  :class="[
                    'grade d-flex align-center',
                    {
                      fail: level.result === 'fail',
                      pass: level.result === 'pass',
                    },
                  ]"
                >
                  {{ level.result }}
                  <Icon :icon="icons[level.result]" />
                </span>
                <span class="level-label">
                  {{ level.level }}
                </span>
              </div>
              <span class="ratio">Ratio: {{ data.ratio }}</span>
            </div>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>
