<template>
  <div id="app">
    <section class="todoapp">
      <header class="header">
        <h1>todos</h1>
        <input
          class="new-todo"
          autofocus
          autocomplete="off"
          placeholder="What needs to be done?"
          v-model="newTodo"
          @keyup.enter="addTodo"
        />
      </header>
      <section class="main" v-show="todos.length" v-cloak>
        <input
          id="toggle-all2"
          class="toggle-all"
          type="checkbox"
          v-model="allDone"
        /><label for="toggle-all2"></label>
        <ul class="todo-list">
          <li
            class="todo"
            v-for="todo in filteredTodos"
            :key="todo.id"
            :class="{ completed: todo.completed, editing: todo == editedTodo }"
          >
            <div class="view">
              <input
                class="toggle"
                type="checkbox"
                v-model="todo.completed"
              /><label @dblclick="editTodo(todo)">{{ todo.title }}</label
              ><button class="destroy" @click="removeTodo(todo)"></button>
            </div>
            <input
              class="edit"
              type="text"
              v-model="todo.title"
              v-todo-focus="todo == editedTodo"
              @blur="doneEdit(todo)"
              @keyup.enter="doneEdit(todo)"
              @keyup.esc="cancelEdit(todo)"
            />
          </li>
        </ul>
      </section>
      <footer class="footer" style="">
        <span class="todo-count"
          ><strong>{{ remaining }}</strong> items left
        </span>
        <ul class="filters">
          <li>
            <a href="#/all" :class="{ selected: visibility == 'all' }">All</a>
          </li>
          <li>
            <a href="#/active" :class="{ selected: visibility == 'active' }"
              >Active</a
            >
          </li>
          <li>
            <a
              href="#/completed"
              :class="{ selected: visibility == 'completed' }"
              >Completed</a
            >
          </li>
        </ul>
        <button
          class="clear-completed"
          style=""
          @click="removeCompleted"
          v-show="todos.length > remaining"
        >
          Clear completed
        </button>
      </footer>
    </section>
    <footer class="info">
      <p>Double-click to edit a todo</p>
      <p>Written by <a href="http://evanyou.me">Evan You</a></p>
      <p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
    </footer>
  </div>
</template>

<script>
const STORAGE_KEY = "todos-vuejs-3.0";
const todoStorage = {
  fetch() {
    const todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
    todos.forEach((todo, index) => {
      todo.id = index;
    });
    todoStorage.uid = todos.length;
    return todos;
  },
  save(todos) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
  },
};
const filters = {
  all(todos) {
    return todos;
  },
  active(todos) {
    return todos.filter((todo) => !todo.completed);
  },
  completed(todos) {
    return todos.filter((todo) => todo.completed);
  },
};

export default {
  data: function () {
    return {
      todos: todoStorage.fetch(),
      newTodo: "",
      editedTodo: null,
      visibility: "",
    };
  },
  watch: {
    todos: {
      handler: function (todos) {
        todoStorage.save(todos);
      },
      deep: true,
    },
  },
  computed: {
    filteredTodos: {
      get() {
        return filters[this.visibility](this.todos);
      },
    },
    remaining() {
      return filters.active(this.todos).length;
    },
    allDone: {
      get() {
        return this.remaining === 0;
      },
      set(value) {
        this.todos.forEach((todo) => {
          todo.completed = value;
        });
      },
    },
  },
  methods: {
    addTodo() {
      // TODO: id
      var title = this.newTodo.trim();
      if (!title) {
        return;
      }
      const todo = {
        id: todoStorage.uid++,
        title: title,
        completed: false,
      };
      this.todos.push(todo);
      this.newTodo = "";
    },
    removeTodo(todo) {
      console.debug("removeTodo");
      this.todos.splice(this.todos.indexOf(todo), 1);
    },
    editTodo(todo) {
      console.debug("editTodo");
      this.beforeEditCache = todo.title;
      this.editedTodo = todo;
    },
    doneEdit(todo) {
      console.debug("doneEditTodo");
      this.editedTodo = null;
      todo.title = todo.title.trim();
      if (!todo.title) {
        this.removeTodo(todo);
      }
    },
    cancelEdit(todo) {
      console.debug("cancelTodo");
      this.editedTodo = null;
      todo.title = this.beforeEditCache;
    },
    removeCompleted() {
      this.todos = filters.active(this.todos);
    },
    onHashChange() {
      const visibility = window.location.hash.replace(/#\/?/, "");
      if (filters[visibility]) {
        this.visibility = visibility;
      } else {
        window.location.hash = "";
        this.visibility = "all";
      }
    },
  },
  created: function () {
    window.addEventListener("hashchange", this.onHashChange);
    this.onHashChange();
  },
  unmounted: function () {
    window.removeEventListener("hashchange", this.onHashChange);
  },
  directives: {
    "todo-focus": {
      updated(el, binding) {
        if (binding.value) {
          console.debug("todo-focus");
          el.focus();
        }
      },
    },
  },
};
</script>

<style scoped src="../../node_modules/todomvc-common/base.css">
[v-cloak] {
  display: none;
}
</style>
