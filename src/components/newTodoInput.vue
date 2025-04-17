<template>

  <div class="window active movable" id="newTodoInput">
    <div class="title-bar">
      <div class="title-bar-text">ToDo List</div>
      <div class="title-bar-controls">
        <button aria-label="Minimize"></button>
        <button aria-label="Maximize"></button>
        <button aria-label="Close"></button>
      </div>
    </div>
    <div class="window-body has-space">

      <form @submit.prevent="submit" class="field-row">

        <input ref="inputEl" v-model="newTodo" placeholder="New Todo" autofocus />
        <button>+</button>
      </form>
    </div>
  </div>

</template>

<script>
import { ref, onMounted } from "vue";

export default {
  name: "NewTodoInput",
  emits: ['add-todo'],
  setup(_, { emit }) {
    const newTodo = ref("");
    const inputEl = ref(null); // allows to force focus on the input el every time, autofocus is bad

    function submit() {
      if (newTodo.value !== "") {
        emit('add-todo', newTodo.value);
        newTodo.value = "";
      }
    }

    onMounted(() => {
      inputEl.value && inputEl.value.focus();
    })

    return {
      newTodo,
      submit,
      inputEl
    }
  },
}
</script>

<style></style>
