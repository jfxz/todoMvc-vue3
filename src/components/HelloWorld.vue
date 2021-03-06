<template>
  <section class="todoapp">
    <header class="header">
      <h1>Little Todo</h1>
      <input
        class="new-todo"
        placeholder="What do you want to do"
        v-model="newTodo"
        @keyup.enter="addTodo"
      >
    </header>

    <section id="toggle-all" class="main" v-show="todos.length" >
      <input class="toggle-all" type="checkbox" v-model="allDone">
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li
          v-for="todo in filteredTodos"
          :key="todo.id"
          class="todo"
          :class="todo==editedTodo?'editing':''"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input
            class="edit"
            type="text"
            v-todo-focus="todo == editedTodo"
            v-model="beforeEditCache"
            @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)"
          >
        </li>
      </ul>
    </section>


    <footer class="footer" v-show="todos.length" v-cloak>
      <span class="todo-count">
        <strong>{{ remaining }}</strong>  left
      </span>
      <ul class="filters">
        <li>
          <button
            v-for="btn in btns"
            :key="btn.value"
            :class="visibility==btn.value?'selected':''"
            @click="visibility=btn.value"
          >{{ btn.label }}</button>
        </li>
      </ul>
      <button
        v-show="todos.length > remaining"
        class="clear-completed"
        @click="removeCompleted"
      > Clear completed
      </button>
    </footer>

  </section>

</template>

<script>
import { reactive, toRefs, computed } from "vue";

export default {
  name: 'Todo',

  setup() {
    const state = reactive({
      newTodo: '',
      todos: [
        { id: 1, title: '吃饭', completed: false },
        { id: 2, title: '睡觉', completed: false },
        { id: 3, title: '打豆豆', completed: false },
      ],
      btns: [
        { label: 'All', value: 'all' },
        { label: 'Active', value: 'active' },
        { label: 'Completed', value: 'completed' },
      ],
      editedTodo: null,
      beforeEditCache: '',
      visibility: 'all'
    })


	const filters = {
		all: function (todos) {
			return todos
		},
		active: function (todos) {
			return todos.filter(function (todo) {
				return !todo.completed
			})
		},
		completed: function (todos) {
			return todos.filter(function (todo) {
				return todo.completed
			})
		}
	}

    const remaining = computed(
      () => state.todos.filter(todo => !todo.completed).length
    )

    const allDone = computed({
      get: function() {
        return remaining.value === 0;
      },
      set: function(value) {
        state.todos.forEach(function(todo) {
          todo.completed = value;
        })
      }
    })

    const filteredTodos = computed(() => filters[state.visibility](state.todos))
    // const filteredTodos = computed(() =>  {
    //   return filters[state.visibility](state.todos)
    // })

    // 添加
    function addTodo () {
      const value = state.newTodo && state.newTodo.trim()
      if (!value) {
        return
      }
      state.todos.push(
        {
          id: state.todos.length + 1,
          title: value,
          completed: false
        }
      )
      state.newTodo = ""
    }

    // 删除
    function removeTodo(todo) {
      const index = state.todos.indexOf(todo)
      state.todos.splice(index, 1)
    }

    // 修改
    function editTodo(todo) {
      state.beforeEditCache = todo.title
      state.editedTodo = todo
    }

    // 保存修改
    function doneEdit(todo) {
      if (!state.editedTodo) {
        return
      }
      state.editedTodo = null
      todo.title = state.beforeEditCache.trim()
      if (!todo.title) {
        removeTodo(todo)
      }
    }

    function removeCompleted() {
      state.todos = state.todos.filter(todo => !todo.completed)
    }

    return {
      ...toRefs(state),
      remaining,
      allDone,
      filteredTodos,
      addTodo,
      removeTodo,
      editTodo,
      doneEdit,
      removeCompleted
    }

  },

  directives: {
    "todo-focus": function(el, binding) {
      if (binding.value) {
        el.focus()
      }
    }
  }

}
</script>
