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
        <!-- TODO - Replace this with a new input dialog, either by pressing "a" which brings up a quick entry box, or using a :n {txt} format -->
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
            <th @click.stop="editCell(todo.id, 'text')">
              <template v-if="editingCell.id === todo.id && editingCell.field === 'text'">
                <input class="editing" v-model="todo.description" @blur="stopEditCell" @keyup.esc="stopEditCell"
                  autofocus />
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
import { ref, onMounted, onUnmounted, nextTick } from "vue";

export default {
  name: 'App',
  components: {},
  setup() {
    const newTodo = ref("");
    const selectedTodoId = ref(null);
    const editingCell = ref({ id: null, field: null });
    const initialLoadData = [
      {
        id: Date.now(),
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
      nextTick(() => {
        const input = document.querySelector("input.editing");
        if (input) {
          input.focus();
        }
      });
    }
    function stopEditCell() {
      editingCell.value = { id: null, field: null };
      localStorage.setItem("todos", JSON.stringify(todos.value))
    }

    function moveSelection(direction) {
      if (todos.value.length === 0) return;

      const currentIndex = todos.value.findIndex(todo => todo.id === selectedTodoId.value);

      let newIndex = currentIndex === -1
        ? 0
        : Math.max(0, Math.min(todos.value.length - 1, currentIndex + direction));
      selectedTodoId.value = todos.value[newIndex].id;
    }

    function handleKey(e) {
      const targetTag = e.target.tagName.toLowerCase();
      const isTyping = targetTag === "input" || targetTag === "textarea";

      if (e.key === "i") {
        if (!isTyping) {
          e.preventDefault();
          console.log(targetTag, isTyping);
          editCell(selectedTodoId.value, 'text');
        }
      }

      if (e.key === "Escape") {
        if (editingCell.value.id !== null) {
          stopEditCell();
        } else {
          selectedTodoId.value = 0;
        }
        return;
      }

      if (isTyping) return;
      if (editingCell.value.id !== null) return;

      if (e.key === "ArrowDown" || e.key === "j") {
        e.preventDefault();
        moveSelection(1);
      } else if (e.key === "ArrowUp" || e.key === "k") {
        e.preventDefault();
        moveSelection(-1);
      }



    }

    onMounted(() => {
      window.addEventListener("keydown", handleKey);
    });

    onUnmounted(() => {
      window.removeEventListener("keydown", handleKey);
    });

    return {
      addTodos, todos, newTodo, selectTodo, selectedTodoId, editCell, stopEditCell, editingCell, moveSelection, handleKey
    }
  },
}
</script>

<style>
.selected {
  background-color: lightblue;
}
</style>
