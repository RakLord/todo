<template>

  <div class="window active" id="appContainer">
    <div class="title-bar">
      <div class="title-bar-text">ToDo List</div>
      <div class="title-bar-controls">
        <button aria-label="Minimize"></button>
        <button aria-label="Maximize"></button>
        <button aria-label="Close"></button>
      </div>
    </div>
    <div class="window-body has-space">

      <form @submit.prevent="addTodos()" class="field-row">
        <input v-model="newTodo" />
        <button>+</button>
      </form>

      <table id="todoList">
        <thead>
          <tr>
            <th>ID</th>
            <th>Description</th>
            <th>Priority</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(todo, index) in todos" :key="todo.id" :class="{ selected: selectedTodoId === todo.id }"
            @click="selectTodo(todo.id)">

            <th>{{ todo.id }}</th>
            <th @click="editCell(todo.id, 'text')">
              <template v-if="editingCell.id === todo.id && editingCell.field === 'text'">
                <input v-model="todo.description" @blur="stopEditCell" @keyup.enter="stopEditCell" autofocus />
              </template>
              <template v-else>
                {{ todo.description }}
              </template>
            </th>
            <th>{{ todo.priority }}</th>
          </tr>
        </tbody>
      </table>

    </div>
  </div>

</template>

<script>
import { ref } from "vue";

export default {
  name: 'App',
  components: {},
  setup() {
    const newTodo = ref("");
    const selectedTodoId = ref(null);
    const editingCell = ref({ id: null, field: null });
    const initialLoadData = [
      {
        id: 1,
        description: "Make ToDo work",
        priority: 4
      },
    ];

    let storedTodos;

    localStorage.getItem("todos")
      ? storedTodos = JSON.parse(localStorage.getItem("todos"))
      : (storedTodos = initialLoadData);

    const todos = ref(storedTodos);

    function addTodos() {
      if (newTodo.value !== "") {
        const newId = Date.now();
        todos.value.push({
          id: newId,
          description: newTodo.value,
          priority: 4

        });
      }
      newTodo.value = "";
    }

    function selectTodo(id) {
      selectedTodoId.value = id;
    }

    function editCell(id, field) {
      editingCell.value = { id, field };
    }
    function stopEditCell() {
      editingCell.value = { id: null, field: null };
    }

    return {
      addTodos, todos, newTodo, selectTodo, selectedTodoId, editCell, stopEditCell, editingCell
    }
  },
}
</script>

<style>
.selected {
  background-color: lightblue;
}
</style>
