<template>
  <!-- Show login form if not logged in -->
  <div v-if="!isLoggedIn" class="container">
    <el-card>
      <h2>Login</h2>
      <el-form @submit.prevent="login">
        <el-form-item label="Username">
          <el-input v-model="username" placeholder="Enter username" clearable />
        </el-form-item>
        <el-form-item label="Password">
          <el-input
            v-model="password"
            type="password"
            placeholder="Enter password"
            show-password
          />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" native-type="submit">Login</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>

  <!-- Show todo app if logged in -->
  <el-card v-else class="todo-app container">
    <div
      style="display: flex; justify-content: space-between; align-items: center"
    >
      <h2>Todo List</h2>
      <el-button type="danger" @click="logout">Logout</el-button>
    </div>

    <el-form @submit.prevent="addTodo" inline>
      <el-form-item>
        <el-input v-model="newTodo" placeholder="New todo" clearable />
      </el-form-item>
      <el-form-item>
        <el-date-picker
          v-model="newDueDate"
          type="date"
          placeholder="Due date"
          format="YYYY-MM-DD"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" native-type="submit">Add</el-button>
      </el-form-item>
    </el-form>

    <el-select
      v-model="sortCriteria"
      placeholder="Sort By"
      @change="sortTodos"
      style="margin: 20px 0"
    >
      <el-option label="Due Date" value="dueDate" />
      <el-option label="Completion" value="completed" />
    </el-select>

    <div
      v-for="(todo, idx) in todos"
      :key="idx"
      class="todo-item"
      :class="{ overdue: isOverdue(todo), completed: todo.completed }"
    >
      <el-checkbox v-model="todo.completed" @change="saveTodos" />
      <span class="todo-text">{{ todo.text }}</span>
      <el-tag
        v-if="todo.dueDate"
        :type="isOverdue(todo) ? 'danger' : 'info'"
        style="margin-left: auto"
      >
        {{ todo.dueDate }}
      </el-tag>
      <div class="todo-actions">
        <button class="icon-button" @click="editTodoItem(idx)">
          <svg
            fill="var(--secondary-color)"
            xmlns="http://www.w3.org/2000/svg"
            height="24px"
            viewBox="0 -960 960 960"
            width="24px"
          >
            <path
              d="M200-200h57l391-391-57-57-391 391v57Zm-80 80v-170l528-527q12-11 26.5-17t30.5-6q16 0 31 6t26 18l55 56q12 11 17.5 26t5.5 30q0 16-5.5 30.5T817-647L290-120H120Zm640-584-56-56 56 56Zm-141 85-28-29 57 57-29-28Z"
            />
          </svg>
        </button>

        <button
          id="delete-btn"
          class="icon-button"
          @click="deleteTodoItem(idx)"
        >
          <svg
            fill="var(--secondary-color)"
            xmlns="http://www.w3.org/2000/svg"
            height="24"
            viewBox="0 -960 960 960"
            width="24"
          >
            <path
              d="M280-120q-33 0-56.5-23.5T200-200v-520h-40v-80h200v-40h240v40h200v80h-40v520q0 33-23.5 56.5T680-120H280Zm400-600H280v520h400v-520ZM360-280h80v-360h-80v360Zm160 0h80v-360h-80v360ZM280-720v520-520Z"
            />
          </svg>
        </button>
      </div>
    </div>
  </el-card>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import {
  ElCard,
  ElForm,
  ElFormItem,
  ElInput,
  ElDatePicker,
  ElButton,
  ElSelect,
  ElOption,
  ElCheckbox,
  ElTag,
  ElButtonGroup,
} from "element-plus";

/* --------------------------------------
  LOGIN FUNCTIONALITY
-------------------------------------- */
const isLoggedIn = ref(false);
const username = ref("");
const password = ref("");
const mockUser = { username: "test", password: "1234" };

const login = () => {
  if (
    username.value === mockUser.username &&
    password.value === mockUser.password
  ) {
    isLoggedIn.value = true;
    localStorage.setItem("loggedIn", "true");
  } else {
    alert('Invalid credentials. Try "test" and "1234"');
  }
};

const logout = () => {
  isLoggedIn.value = false;
  localStorage.removeItem("loggedIn");
};

/* Check login status on page load */
onMounted(() => {
  isLoggedIn.value = localStorage.getItem("loggedIn") === "true";
});

/* --------------------------------------
  TODO FUNCTIONALITY
-------------------------------------- */
const newTodo = ref("");
const newDueDate = ref(null);
const sortCriteria = ref("");
const todos = ref([]);

const saveTodos = () =>
  localStorage.setItem("todos", JSON.stringify(todos.value));
const getTodos = () =>
  (todos.value = JSON.parse(localStorage.getItem("todos") || "[]"));

onMounted(getTodos);
watch(todos, saveTodos, { deep: true });

const addTodo = () => {
  if (!newTodo.value.trim()) return alert("Please enter a todo");
  todos.value.push({
    text: newTodo.value.trim(),
    dueDate: newDueDate.value,
    completed: false,
  });
  newTodo.value = "";
  newDueDate.value = null;
};

const deleteTodoItem = (idx) => todos.value.splice(idx, 1);

const editTodoItem = (idx) => {
  const current = todos.value[idx];
  const newText = prompt("Edit todo:", current.text);
  const newDate = prompt("Edit due date (YYYY-MM-DD):", current.dueDate);
  if (newText?.trim()) {
    current.text = newText.trim();
    current.dueDate = newDate || "";
  }
};

const sortTodos = () => {
  if (sortCriteria.value === "dueDate") {
    todos.value.sort(
      (a, b) => new Date(a.dueDate || 0) - new Date(b.dueDate || 0)
    );
  } else if (sortCriteria.value === "completed") {
    todos.value.sort((a, b) => a.completed - b.completed);
  }
};

const isOverdue = (todo) =>
  todo.dueDate && new Date(todo.dueDate) < new Date() && !todo.completed;
</script>

<style>
:root {
  --background: #101114;
  --primary-color: #1c1d20;
  --secondary-color: #4a4d57;
  --accent-color: #7ad9e6;
  --text-color: #f9f9f9;
  --danger-color: #ff0033;
}

body {
  background-color: var(--background);
  color: var(--text-color);
  font-family: "Segoe UI", sans-serif;
}

.container {
  max-width: 800px;
  margin: 40px auto;
  background: var(--primary-color);
  border-radius: 20px;
  padding: 20px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
  color: var(--text-color);
}

h1,
h2 {
  text-align: center;
  color: var(--accent-color);
  margin-bottom: 20px;
  font-size: 2rem;
}

/* Element Plus Overrides */
::v-deep(.el-card) {
  background-color: var(--primary-color);
  border-radius: 20px;
  color: var(--text-color);
  padding: 20px;
}

::v-deep(.el-input__inner),
::v-deep(.el-textarea__inner),
::v-deep(.el-select .el-input__inner),
::v-deep(.el-date-editor .el-input__inner) {
  background-color: var(--background);
  border: 2px solid var(--secondary-color);
  color: var(--text-color);
  border-radius: 12px;
}

::v-deep(.el-form-item__label) {
  color: var(--accent-color);
}

::v-deep(.el-button) {
  border-radius: 12px;
  font-weight: bold;
}

::v-deep(.el-button--primary) {
  background-color: var(--accent-color);
  border: none;
  color: var(--primary-color);
}

::v-deep(.el-button--primary:hover) {
  background-color: #66c5d5;
}

::v-deep(.el-button--danger) {
  background-color: var(--danger-color);
  color: white;
}

::v-deep(.el-select-dropdown),
::v-deep(.el-picker-panel) {
  background-color: var(--primary-color);
  color: var(--text-color);
}

.icon-button {
  background: none;
  border: none;
  padding: 4px;
  cursor: pointer;
  transition: transform 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
}

.icon-button:hover {
  transform: scale(1.1);
  background-color: var(--accent-color);
}

#delete-btn:hover {
  background-color: var(--danger-color);
  transform: scale(1.1);
}

.todo-text:hover {
  cursor: pointer;
}

/* Tag Styling */
::v-deep(.el-tag.el-tag--info) {
  color: var(--primary-color);
  border: none;
}

::v-deep(.el-tag.el-tag--danger) {
  background-color: var(--danger-color);
  color: white;
  border: none;
}

/* Todo Item Styling */
.todo-item {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 12px;
  background-color: var(--primary-color);
  border: 1px solid var(--secondary-color);
  border-radius: 12px;
  padding: 10px 15px;
  transition: background 0.3s;
}

.todo-item:hover {
  background-color: #26272b;
}

.todo-text {
  flex: 1;
  font-size: 1rem;
}

.completed .todo-text {
  text-decoration: line-through;
  color: var(--secondary-color);
}

.overdue .todo-text {
  color: var(--danger-color);
  font-weight: bold;
}

/* Responsive Form Adjustments */
@media (max-width: 600px) {
  ::v-deep(.el-form) {
    flex-direction: column;
  }

  ::v-deep(.el-form-item) {
    width: 100%;
  }

  ::v-deep(.el-button--primary) {
    width: 100%;
  }
}
</style>
