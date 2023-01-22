<script lang="ts" setup>
import { computed, ref } from "vue";

const themeColoursInput = ref("");
const themeColours: { [key: string]: any } = computed(() => {
  if (themeColoursInput.value === "") {
    return {};
  }

  const data: { [key: string]: any } = {};
  const jsonData = JSON.parse(themeColoursInput.value);
  Object.keys(jsonData).forEach((colour: string) => {
    console.log("color", colour);
    data[colour] = {
      name: colour,
      colour: jsonData[colour],
      // ratio:
    };
  });
  console.log("data", data);

  return data;
});

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
    <div
      v-for="(data, colour) in themeColours"
      :key="colour"
      class="d-flex column"
    >
      <div>
        <span>{{ colour }}</span
        ><span>Ratio: {{ data.ratio }}</span>
      </div>
      <div class="levels">
        <!-- <div
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
        </div> -->
      </div>
    </div>
    <h2>Theme Checker</h2>
    <p>
      Paste a json colour theme below to check the colour contrast for each
      colour.
    </p>
    <label for="themeColours">Theme colours:</label>
    <textarea
      v-model="themeColoursInput"
      id="themeColours"
      name="themeColours"
      rows="10"
      @keyup="autoGrow"
    ></textarea>
  </div>
</template>
