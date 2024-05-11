<template>
  <div>
    <form @submit.prevent="addTodo()">
      <center>
        <h2 @click="editTodoListName">
          <span v-if="!editingTodoListName">{{ todoListName }}</span>
          <input
            v-model="editedTodoListName"
            v-if="editingTodoListName"
            @keyup.enter="saveTodoListName()"
            @blur="saveTodoListName()"
            @keyup.escape="cancelEditingTodoListName"
            @keyup.shift.enter="cancelEditingTodoListName"
          />
        </h2>
      </center>
      <label>New ToDo </label>
      <input v-model="newTodo" name="newTodo" autocomplete="off" />
      <button>Add ToDo</button>
    </form>

    <div class="filter-buttons">
      <button
        class="filter-button"
        :class="{ active: filter === 'all' }"
        @click="filter = 'all'"
      >
        All
      </button>
      <button
        class="filter-button"
        :class="{ active: filter === 'done' }"
        @click="filter = 'done'"
      >
        Done
      </button>
      <button
        class="filter-button"
        :class="{ active: filter === 'undone' }"
        @click="filter = 'undone'"
      >
        Undone
      </button>
    </div>

    <ul>
      <li v-for="(todo, index) in filteredTodos" :key="index">
        <input
          type="checkbox"
          v-model="todo.done"
          :id="'todo_' + index"
        />
        <label :for="'todo_' + index" :class="{ done: todo.done }">
          <span v-if="!todo.editing">{{ todo.content }}</span>
          <input
            v-else
            v-model="todo.content"
            @keyup.enter="saveEditedTodo(index)"
            @blur="saveEditedTodo(index)"
            @keyup.escape="cancelEditingTodo(index)"
            @keyup.shift.enter="cancelEditingTodo(index)"
          />
        </label>
        <button @click="removeTodo(index)">Remove</button>
        <button @click="editTodo(index)">
          {{ todo.editing ? "Cancel" : "Edit" }}
        </button>
      </li>
    </ul>
    <h4 v-if="filteredTodos.length === 0">Empty list.</h4>
  </div>
</template>

<script>
import { ref, computed } from "vue";

export default {
  name: "App",
  setup() {
    const newTodo = ref("");
    const defaultData = [
      {
        done: false,
        content: "Write a blog post",
        editing: false,
      },
    ];
    const todosData = JSON.parse(localStorage.getItem("todos")) || defaultData;
    const todos = ref(todosData);
    const filter = ref("all");
    const editingTodoListName = ref(false);
    const todoListName = ref("My Todo List");
    const editedTodoListName = ref(todoListName.value);

    function addTodo() {
      if (newTodo.value) {
        todos.value.push({
          done: false,
          content: newTodo.value,
          editing: false,
        });
        newTodo.value = "";
        saveData();
      }
    }

    function removeTodo(index) {
      todos.value.splice(index, 1);
      saveData();
    }

    function saveData() {
      const storageData = JSON.stringify(todos.value);
      localStorage.setItem("todos", storageData);
    }

    function editTodoListName() {
      editedTodoListName.value = todoListName.value;
      editingTodoListName.value = true;
    }

    function saveTodoListName() {
      todoListName.value = editedTodoListName.value;
      editingTodoListName.value = false;
      saveData();
    }

    function cancelEditingTodoListName() {
      editedTodoListName.value = todoListName.value;
      editingTodoListName.value = false;
    }

    function editTodo(index) {
      const todo = todos.value[index];
      todo.editing = !todo.editing;
      saveData();
    }

    function saveEditedTodo() {
      saveData();
    }

    function cancelEditingTodo(index) {
      todos.value[index].editing = false;
    }

    const filteredTodos = computed(() => {
      if (filter.value === "done") {
        return todos.value.filter((todo) => todo.done);
      } else if (filter.value === "undone") {
        return todos.value.filter((todo) => !todo.done);
      } else {
        return todos.value;
      }
    });

    return {
      todos,
      newTodo,
      addTodo,
      removeTodo,
      saveData,
      filter,
      filteredTodos,
      editingTodoListName,
      todoListName,
      editedTodoListName,
      editTodo,
      saveEditedTodo,
      cancelEditingTodoListName,
      cancelEditingTodo,
    };
  },
};
</script>