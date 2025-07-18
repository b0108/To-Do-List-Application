<template>
  <div class="todo-app container">
    <h1>Todo List</h1>
    <form @submit.prevent="addTodo">
      <input v-model="newTodo" id="todo-input" placeholder="Add a todo..." />
      <input v-model="newDueDate" id="due-date" type="date" />
      <button id="add-button" type="submit">Add</button>
    </form>

    <select id="sort-select" v-model="sortCriteria" @change="sortTodos">
      <option disabled value="">Sort By</option>
      <option value="dueDate">Due Date</option>
      <option value="completed">Completion</option>
    </select>

    <ul id="todo-list">
      <li
        v-for="(todo, index) in todos"
        :key="index"
        :class="[
          'todo',
          { overdue: isOverdue(todo), completed: todo.completed },
        ]"
      >
        <input
          type="checkbox"
          :id="'todo-' + index"
          v-model="todo.completed"
          @change="saveTodos"
        />
        <label class="custom-checkbox" :for="'todo-' + index">
          <svg
            fill="transparent"
            xmlns="http://www.w3.org/2000/svg"
            height="24"
            viewBox="0 -960 960 960"
            width="24"
          >
            <path d="M382-240 154-468l57-57 171 171 367-367 57 57-424 424Z" />
          </svg>
        </label>

        <label :for="'todo-' + index" class="todo-text">{{ todo.text }}</label>
        <span class="todo-due-date">Due: {{ todo.dueDate || "No date" }}</span>

        <button class="edit-button" @click="editTodoItem(index)">
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

        <button class="delete-button" @click="deleteTodoItem(index)">
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
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";

const newTodo = ref("");
const newDueDate = ref("");
const sortCriteria = ref("");
const todos = ref([]);

const saveTodos = () => {
  localStorage.setItem("todos", JSON.stringify(todos.value));
};

const getTodos = () => {
  todos.value = JSON.parse(localStorage.getItem("todos") || "[]");
};

onMounted(() => {
  getTodos();
});

watch(todos, saveTodos, { deep: true });

const addTodo = () => {
  if (newTodo.value.trim() === "") {
    alert("Please add a Todo!");
    return;
  }
  todos.value.push({
    text: newTodo.value.trim(),
    dueDate: newDueDate.value,
    completed: false,
  });
  newTodo.value = "";
  newDueDate.value = "";
};

const deleteTodoItem = (index) => {
  todos.value.splice(index, 1);
};

const editTodoItem = (index) => {
  const current = todos.value[index];
  const newText = prompt("Edit your todo:", current.text);
  const newDate = prompt("Edit due date (YYYY-MM-DD):", current.dueDate);
  if (newText && newText.trim() !== "") {
    current.text = newText.trim();
    current.dueDate = newDate || "";
  } else {
    alert("Todo cannot be empty!");
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

const isOverdue = (todo) => {
  return todo.dueDate && new Date(todo.dueDate) < new Date() && !todo.completed;
};
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
  margin: 0;
  background-color: #e5e5e5;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

.container {
  max-width: 800px;
  margin: 40px auto;
  background: var(--background);
  border-radius: 20px;
  padding: 20px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
  color: var(--text-color);
}

h1 {
  text-align: center;
  color: var(--accent-color);
  margin-bottom: 20px;
  font-size: 2rem;
}

form {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 20px;
}

input,
select,
button {
  font: inherit;
}

#todo-input,
#due-date {
  flex: 1;
  padding: 10px;
  border-radius: 20px;
  border: 2px solid var(--secondary-color);
  background: transparent;
  color: var(--text-color);
  min-width: 120px;
}

#sort-select {
  background-color: var(--primary-color);
  padding: 12px 20px;
  border: 2px solid var(--secondary-color);
  border-radius: 1000px;
  font: inherit;
  color: var(--text-color);
  cursor: pointer;
}

select,
::picker(select) {
  appearance: base-select;
}

select::picker-icon {
  color: var(--secondary-color);
  transition: 0.4s rotate;
}

select:open::picker-icon {
  rotate: 180deg;
}

::picker(select) {
  border: none;
  border-radius: 20px;
  border-color: var(--primary-color);
}
/* 
#sort-select.option:hover {
  background-color: var(--accent-color);
  cursor: pointer;
  color: var(--primary-color);
} */

#add-button {
  background: var(--accent-color);
  border: none;
  border-radius: 20px;
  padding: 10px 20px;
  color: var(--primary-color);
  font-weight: bold;
  cursor: pointer;
}

#add-button:hover {
  background: #66c5d5;
}

.todo {
  display: flex;
  align-items: center;
  gap: 10px;
  background: var(--primary-color);
  padding: 10px;
  border-radius: 15px;
  margin-bottom: 10px;
  flex-wrap: wrap;
}

.todo-text {
  flex-grow: 1;
  cursor: pointer;
}

.todo-due-date {
  font-size: 0.75rem;
  color: var(--accent-color);
  margin-left: auto;
}

.todo.overdue .todo-text {
  color: var(--danger-color);
  font-weight: bold;
}

input[type="checkbox"] {
  display: none;
}

.custom-checkbox {
  border: 2px solid var(--accent-color);
  border-radius: 50%;
  min-height: 20px;
  min-width: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

input[type="checkbox"]:checked ~ .custom-checkbox {
  background: var(--accent-color);
}

input[type="checkbox"]:checked ~ .custom-checkbox svg {
  fill: var(--primary-color);
}

input[type="checkbox"]:checked ~ .todo-text {
  text-decoration: line-through;
  color: var(--secondary-color);
}

.edit-button,
.delete-button {
  background: none;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.edit-button svg,
.delete-button svg {
  transition: 0.2s ease;
  fill: var(--accent-color);
}

.edit-button:hover svg,
.delete-button:hover svg {
  fill: var(--danger-color);
}

@media (max-width: 600px) {
  form {
    flex-direction: column;
  }

  #add-button {
    width: 100%;
  }
}
</style>
