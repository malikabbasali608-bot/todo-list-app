const STORAGE_KEY = "taskflow.tasks.v1";

const todoForm = document.getElementById("todoForm");
const todoInput = document.getElementById("todoInput");
const taskList = document.getElementById("taskList");
const totalCountEl = document.getElementById("totalCount");
const remainingCountEl = document.getElementById("remainingCount");
const completedCountEl = document.getElementById("completedCount");
const filterButtons = document.querySelectorAll(".filter-button");
const clearCompletedBtn = document.getElementById("clearCompletedBtn");
const taskTemplate = document.getElementById("taskTemplate");

const state = {
  tasks: loadTasks(),
  filter: "all",
};

function loadTasks() {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (!saved) {
      return [
        {
          id: crypto.randomUUID(),
          text: "Welcome to TaskFlow! Add your first task.",
          completed: false,
          createdAt: new Date().toISOString(),
        },
      ];
    }

    const parsed = JSON.parse(saved);
    return Array.isArray(parsed) ? parsed : [];
  } catch (error) {
    console.error("Failed to load tasks from localStorage:", error);
    return [];
  }
}

function saveTasks() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state.tasks));
}

function formatDate(isoString) {
  if (!isoString) return "Just now";

  const date = new Date(isoString);
  return new Intl.DateTimeFormat("en-US", {
    month: "short",
    day: "numeric",
    hour: "numeric",
    minute: "2-digit",
  }).format(date);
}

function updateStats() {
  const total = state.tasks.length;
  const completed = state.tasks.filter((task) => task.completed).length;
  const remaining = total - completed;

  totalCountEl.textContent = String(total);
  remainingCountEl.textContent = String(remaining);
  completedCountEl.textContent = String(completed);
}

function getVisibleTasks() {
  if (state.filter === "active") {
    return state.tasks.filter((task) => !task.completed);
  }

  if (state.filter === "completed") {
    return state.tasks.filter((task) => task.completed);
  }

  return state.tasks;
}

function renderTasks() {
  const visibleTasks = getVisibleTasks();

  if (!visibleTasks.length) {
    taskList.innerHTML = '<li class="empty-state">No tasks here yet. Add one to get started.</li>';
    updateStats();
    return;
  }

  taskList.innerHTML = "";

  visibleTasks.forEach((task) => {
    const fragment = taskTemplate.content.cloneNode(true);
    const item = fragment.querySelector(".task-item");
    const checkbox = fragment.querySelector(".task-checkbox");
    const text = fragment.querySelector(".task-text");
    const meta = fragment.querySelector(".task-meta");
    const editButton = fragment.querySelector(".edit-button");
    const deleteButton = fragment.querySelector(".delete-button");

    checkbox.checked = task.completed;
    text.textContent = task.text;
    meta.textContent = formatDate(task.createdAt);

    if (task.completed) {
      item.classList.add("completed");
    }

    checkbox.dataset.id = task.id;
    checkbox.addEventListener("change", (event) => {
      toggleTask(event.target.dataset.id);
    });

    editButton.dataset.id = task.id;
    editButton.addEventListener("click", () => editTask(task.id));

    deleteButton.dataset.id = task.id;
    deleteButton.addEventListener("click", () => deleteTask(task.id));

    taskList.appendChild(fragment);
  });

  updateStats();
}

function addTask(text) {
  const trimmed = text.trim();
  if (!trimmed) {
    todoInput.focus();
    return;
  }

  state.tasks.unshift({
    id: crypto.randomUUID(),
    text: trimmed,
    completed: false,
    createdAt: new Date().toISOString(),
  });

  saveTasks();
  renderTasks();
}

function toggleTask(id) {
  state.tasks = state.tasks.map((task) =>
    task.id === id ? { ...task, completed: !task.completed } : task
  );

  saveTasks();
  renderTasks();
}

function deleteTask(id) {
  state.tasks = state.tasks.filter((task) => task.id !== id);
  saveTasks();
  renderTasks();
}

function clearCompletedTasks() {
  state.tasks = state.tasks.filter((task) => !task.completed);
  saveTasks();
  renderTasks();
}

function editTask(id) {
  const task = state.tasks.find((item) => item.id === id);
  if (!task) return;

  const nextValue = window.prompt("Edit task", task.text);
  if (nextValue === null) return;

  const trimmedValue = nextValue.trim();
  if (!trimmedValue) {
    window.alert("Task cannot be empty.");
    return;
  }

  state.tasks = state.tasks.map((item) =>
    item.id === id ? { ...item, text: trimmedValue } : item
  );

  saveTasks();
  renderTasks();
}

todoForm.addEventListener("submit", (event) => {
  event.preventDefault();
  addTask(todoInput.value);
  todoInput.value = "";
  todoInput.focus();
});

filterButtons.forEach((button) => {
  button.addEventListener("click", () => {
    state.filter = button.dataset.filter;

    filterButtons.forEach((btn) => {
      btn.classList.toggle("active", btn === button);
    });

    renderTasks();
  });
});

clearCompletedBtn.addEventListener("click", () => {
  clearCompletedTasks();
});

renderTasks();
