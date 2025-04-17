<template>

  <HelpBox style="z-index: 99; max-width: 600px;"
    :style="{ left: windowPositions.help.x + 'px', top: windowPositions.help.y + 'px' }" v-if="displayHelp === true"
    @mousedown="startDrag($event, 'help')" />

  <NewTodoInput @add-todo="handleNewTodo" v-if="creatingTodo === true" style="z-index: 20;"
    :style="{ left: windowPositions.input.x + 'px', top: windowPositions.input.y + 'px' }"
    @mousedown="startDrag($event, 'input')" />

  <div class="window active movable" id="todoList" style="min-width: 30%;max-width: 600px; z-index: 10;"
    :style="{ left: windowPositions.list.x + 'px', top: windowPositions.list.y + 'px' }"
    @mousedown="startDrag($event, 'list')">
    <div class="title-bar">
      <div class="title-bar-text">ToDo List</div>
      <div class="title-bar-controls">
        <button aria-label="Minimize"></button>
        <button aria-label="Maximize"></button>
        <button aria-label="Close"></button>
      </div>
    </div>
    <div class="window-body has-space">


      <table id="todoList">
        <thead>
          <tr>
            <th>ID</th>
            <th>Description</th>
            <th>Priority</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="todo in todos" :key="todo.id" :class="{ selected: selectedTodoId === todo.id }"
            @click="selectTodo(todo.id)">

            <td>{{ todo.id }}</td>

            <td @click.stop="editCell(todo.id, 'text')" class="description-cell">
              <template v-if="editingCell.id === todo.id && editingCell.field === 'text'">
                <input class="editing" v-model="todo.description" @blur="stopEditCell" @keyup.esc="stopEditCell"
                  @keyup.enter="stopEditCell" autofocus />
              </template>
              <template v-else>
                {{ todo.description }}
              </template>
            </td>

            <td>{{ todo.priority }}</td>
          </tr>
        </tbody>
      </table>

    </div>
  </div>


</template>

<script>
import { ref, onMounted, onUnmounted, nextTick, watch } from "vue";
import NewTodoInput from "./components/newTodoInput.vue";
import HelpBox from "./components/help.vue";

export default {
  name: 'App',
  components: { NewTodoInput, HelpBox },
  setup() {
    const newTodo = ref("");
    const selectedTodoId = ref(null);
    const prevSelectedTodoId = ref(null);
    const editingCell = ref({ id: null, field: null });

    const creatingTodo = ref(false);
    const displayHelp = ref(true);


    const defaultWindowPositions = {
      list: { x: 50, y: 50 },
      input: { x: 300, y: 100 },
      help: { x: 10, y: 10 },
    };

    let saved = {}
    try {
      saved = JSON.parse(localStorage.getItem("windowPos")) || {}
    } catch (_) {
      //ignore invalid JSON
    }

    // Rebuild the ref (needed for if we add new windows later)
    function mergePositions(def, sv) {
      const out = {}
      for (const key in def) {
        out[key] = {
          ...def[key],
          ...(sv[key] || {})
        }
      }
      return out
    }

    const windowPositions = ref(
      mergePositions(defaultWindowPositions, saved)
    )
    const drag = ref(null);

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

    function save() {
      localStorage.setItem("todos", JSON.stringify(todos.value));
    }

    function startDrag(evt, id) {
      console.log(id);
      if (evt.button !== 0) return;
      drag.value = {
        id,
        startX: evt.clientX,
        startY: evt.clientY,
        origX: windowPositions.value[id].x,
        origY: windowPositions.value[id].y,
      };
      window.addEventListener("mousemove", onDrag);
      window.addEventListener("mouseup", endDrag);
    }

    function onDrag(evt) {
      if (!drag.value) return;
      const dx = evt.clientX - drag.value.startX;
      const dy = evt.clientY - drag.value.startY;

      /* update the reactive store → DOM moves automatically */
      const pos = windowPositions.value[drag.value.id];
      pos.x = drag.value.origX + dx;
      pos.y = drag.value.origY + dy;
    }

    function endDrag() {
      if (!drag.value) return;
      drag.value = null;
      window.removeEventListener('mousemove', onDrag);
      window.removeEventListener('mouseup', endDrag);
    }

    watch(
      windowPositions,
      (val) => localStorage.setItem('windowPos', JSON.stringify(val)),
      { deep: true }
    );

    function handleNewTodo(text) {
      const newId = Date.now();
      todos.value.push({
        id: newId,
        description: text,
        priority: 4,
      });
      save();
    }

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
      save();
    }

    function moveSelection(direction) {
      if (todos.value.length === 0) return;

      const currentIndex = todos.value.findIndex(todo => todo.id === selectedTodoId.value);
      if (selectedTodoId) {

        prevSelectedTodoId.value = selectedTodoId.value;
      }
      let newIndex = currentIndex === -1
        ? 0
        : Math.max(0, Math.min(todos.value.length - 1, currentIndex + direction));
      selectedTodoId.value = todos.value[newIndex].id;
    }

    function deleteSelectedTodo() {
      if (selectedTodoId.value === null) return;
      todos.value = todos.value.filter(todo => todo.id !== selectedTodoId.value);
      selectedTodoId.value = prevSelectedTodoId.value;
      save();
    }

    function createTodo() {
      creatingTodo.value = true;
    }

    function handleKey(e) {
      const targetTag = e.target.tagName.toLowerCase();
      let isTyping;
      isTyping = targetTag === "input" || targetTag === "textarea";
      if (e.key === "Escape") {
        isTyping = false;
      }

      if (e.key === "a") {
        // Bring up new todo input
        if (isTyping) return;
        createTodo();
      }
      if (e.key === "i") {
        if (isTyping) return;
        e.preventDefault();
        console.log(targetTag, isTyping);
        editCell(selectedTodoId.value, 'text');
      }

      if (e.key === "Escape") {
        if (creatingTodo.value === true) {
          creatingTodo.value = false;
        }
        if (editingCell.value.id !== null) {
          stopEditCell();
        } else {
          selectedTodoId.value = 0;
        }
        return;
      }

      if (isTyping) return; // =============== Anything after this does not run if in an input box
      if (editingCell.value.id !== null) return;

      if (e.key === "ArrowDown" || e.key === "j") {
        e.preventDefault();
        moveSelection(1);
      } else if (e.key === "ArrowUp" || e.key === "k") {
        e.preventDefault();
        moveSelection(-1);
      }

      if (e.key === "d") {
        e.preventDefault();
        deleteSelectedTodo();
      }

      if (e.key === "m") {
        e.preventDefault();
        windowPositions.value = defaultWindowPositions;
      }
      if (e.key === "h") {
        e.preventDefault();
        displayHelp.value = !displayHelp.value;
      }



    }

    onMounted(() => {
      window.addEventListener("keydown", handleKey);
    });

    onUnmounted(() => {
      window.removeEventListener("keydown", handleKey);
    });

    return {
      addTodos, todos, newTodo, selectTodo, selectedTodoId, editCell, stopEditCell, editingCell, moveSelection, handleKey, handleNewTodo, deleteSelectedTodo, creatingTodo, createTodo, windowPositions, startDrag, displayHelp
    }
  },
}
</script>

<style>
table {
  width: 100%;
}

.selected {
  background-color: lightblue;
}

.description-cell {
  max-width: 40ch;
  white-space: normal;
  word-break: break-word;
  overflow-wrap: anywhere;
}
</style>
