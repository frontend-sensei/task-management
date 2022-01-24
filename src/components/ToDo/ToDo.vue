<template>
  <section class="todo">
    <header class="todo__header">
      <input
        class="todo__input"
        autofocus
        placeholder="Create task"
        v-model="newTodo"
        @keyup.enter="addTodo"
      />
    </header>
    <section class="todo__main" v-show="todos.length">
      <ul class="todo-list">
        <li
          v-for="todo in filteredTodos"
          class="todo-item"
          :key="todo.id"
          :class="{ completed: todo.completed }"
        >
          <input
            class="todo-item__toggle-btn"
            type="checkbox"
            v-model="todo.completed"
          />
          <label class="todo-item__label" @dblclick="editTodo(todo)">
            {{ todo.title }}
          </label>
          <button class="todo-item__remove-btn" @click="removeTodo(todo)">
            ×
          </button>
        </li>
      </ul>
    </section>
    <footer class="todo__footer" v-show="todos.length" v-cloak>
      <span class="todo-count">
        <strong>{{ remaining }}</strong> {{ remaining | pluralize }} left
      </span>
      <ul class="todo__filters">
        <li>
          <button
            @click="changeVisibility('all')"
            :class="{ selected: visibility == 'all' }"
            class="todo-filter"
          >
            All
          </button>
        </li>
        <li>
          <button
            @click="changeVisibility('active')"
            :class="{ selected: visibility == 'active' }"
            class="todo-filter"
          >
            Active
          </button>
        </li>
        <li>
          <button
            @click="changeVisibility('completed')"
            :class="{ selected: visibility == 'completed' }"
            class="todo-filter"
          >
            Completed
          </button>
        </li>
      </ul>
      <button
        class="todo-btn todo__clear-completed-btn"
        @click="removeCompleted"
        v-show="todos.length > remaining"
      >
        Clear completed
      </button>
    </footer>
  </section>
</template>

<script>
import { todoStorage } from "./storage";
import { filters } from "./filters";

export default {
  name: "ToDo",
  data() {
    return {
      todos: todoStorage.fetch(),
      newTodo: "",
      editedTodo: null,
      visibility: "all",
    };
  },
  filters: {
    pluralize(n) {
      return n === 1 ? "item" : "items";
    },
  },
  computed: {
    filteredTodos() {
      return filters[this.visibility](this.todos);
    },
    remaining() {
      return filters.active(this.todos).length;
    },
  },
  watch: {
    todos: {
      handler(todos) {
        todoStorage.save(todos);
      },
      deep: true,
    },
  },
  methods: {
    changeVisibility(visibility) {
      if (filters[visibility]) {
        this.visibility = visibility;
      } else {
        this.visibility = "all";
      }
    },

    addTodo() {
      const value = this.newTodo && this.newTodo.trim();
      if (!value) {
        return;
      }
      this.todos.push({
        id: todoStorage.uid++,
        title: value,
        completed: false,
      });
      this.newTodo = "";
    },

    removeTodo(todo) {
      this.todos.splice(this.todos.indexOf(todo), 1);
    },

    editTodo(todo) {
      this.beforeEditCache = todo.title;
      this.editedTodo = todo;
    },

    removeCompleted() {
      this.todos = filters.active(this.todos);
    },
  },
};
</script>

<style scoped lang="scss">
.todo {
  max-width: 500px;
  margin: 3rem auto;
  border-radius: 5px;
  box-shadow: 0 0 4px #c5c5c5;
}
.todo-filter {
  margin: 0 0.25rem;
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  background-color: #eee;
  &.selected {
    background-color: #bfbfbf;
  }
}
.todo__input {
  width: 100%;
  padding: 0.75rem;
  border: none;
  box-shadow: inset 0 0 4px -2px #00000054;
}
.todo-list {
  margin: 0;
  padding: 0;
  list-style-type: none;
}
.todo-item {
  display: flex;
  align-items: center;
  padding: 0.5rem;
}
.todo-item__toggle-btn {
  --size: 20px;
  width: var(--size);
  height: var(--size);
}
.todo-item__label {
  padding-left: 1rem;
  width: 100%;
  text-align: left;
}
.todo-item__remove-btn {
  --size: 20px;
  width: var(--size);
  height: var(--size);
}
.todo__footer {
  padding: 0.5rem;
  display: flex;
  align-items: center;
}
.todo__filters {
  margin: 0 1rem;
  padding: 0;
  list-style-type: none;
  display: flex;
}
.todo__clear-completed-btn {
  margin-left: auto;
}
</style>
