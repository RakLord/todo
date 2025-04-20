<template>
  <div class="window active movable" id="helpBox">
    <div class="title-bar">
      <div class="title-bar-text">Help</div>
      <div class="title-bar-controls">
        <button aria-label="Minimize"></button>
        <button aria-label="Maximize"></button>
        <button aria-label="Close"></button>
      </div>
    </div>

    <div class="window-body has-space">
      <table id="KeybindsTable">
        <thead>
          <tr>
            <th>Key</th>
            <th>Description</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(desc, key) in keybinds" :key="key">
            <td>{{ key }}</td>
            <td>{{ desc }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
<script>
import { ref, onMounted } from "vue";

export default {
  name: "HelpBox",
  setup(_) {

    const HelpBox = ref(null);
    const keybinds = ref({
      'a': 'Add new ToDo',
      'j/k': 'Navigate ToDo list',
      'i': 'Edit selected ToDo text',
      'enter': 'Save ToDo edit',
      'esc': 'Close entry box / escape',
      'd': 'Delete selected ToDo',
      'q/e': 'Edit selected ToDo priority',
      'm': 'Reset window positions (may need refresh first)',
      'h': 'Show/Hide help'
    })
    onMounted(() => {
      console.log("Helped");
    })

    return {
      HelpBox,
      keybinds
    }
  },
}
</script>

<style></style>
