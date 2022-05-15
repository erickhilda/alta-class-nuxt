<script>
import gql from 'graphql-tag'
import todosQuery from '../queries/allTodo.gql'
// import todosUsers from '../queries/allUsers.gql'

export default {
  data: () => ({
    isDone: false,
    newTodo: '',
    withFilter: true,
    selectedTodo: null,
    isError: false,
  }),
  apollo: {
    todos: {
      query: todosQuery,
      variables() {
        return {
          is_done: this.isDone,
        }
      },
      errorPolicy: 'all',
      fetchPolicy: 'cache-and-network',
      error(error) {
        console.log('error', error)
        this.isError = true
      },
      subscribeToMore: {
        document: gql`
          subscription todoAdded($is_done: Boolean) {
            todos(where: { is_done: { _eq: $is_done } }) {
              id
              title
              is_done
              user {
                id
                name
              }
            }
          }
        `,
        variables() {
          return {
            is_done: this.isDone,
          }
        },
        updateQuery: (prev, { subscriptionData }) => {
          const newTodos = subscriptionData.data.todos
          return {
            todos: [...newTodos],
          }
        },
      },
    },
  },
  methods: {
    deleteTodo(id) {
      this.$apollo.mutate({
        mutation: gql`
          mutation deleteTodo($id: Int!) {
            delete_todos(where: { id: { _eq: $id } }) {
              returning {
                id
              }
            }
          }
        `,
        variables: { id },
      })
    },
    addTodo() {
      this.$apollo.mutate({
        mutation: gql`
          mutation addTodo($id: Int!, $title: String!) {
            insert_todos_one(object: { user_id: $id, title: $title }) {
              id
              title
              is_done
            }
          }
        `,
        variables: {
          id: 3,
          title: this.newTodo,
        },
        updateQueries: {
          todosQuery: (prev, { mutationResult }) => {
            const newTodo = mutationResult.data.insert_todos_one
            const newTodos = [newTodo, ...prev.todos]
            return {
              todos: newTodos,
            }
          },
        },
        optimisticResponse: {
          __typename: 'Mutation',
          insert_todos_one: {
            __typename: 'todos',
            id: -1,
            title: this.newTodo,
            is_done: false,
          },
        },
      })
    },
    editTodo(todo) {
      this.selectedTodo = todo
    },
    refetchTodos() {
      this.$apollo.queries.todos.refetch()
    },
  },
}
</script>

<template>
  <div id="app">
    <form @submit.prevent="addTodo">
      <input
        v-model="newTodo"
        placeholder="What need to be done"
        autocomplete="off"
      />
      <button type="submit">Add Todo</button>
    </form>
    <div>
      <label>
        <input v-model="isDone" type="radio" :value="true" />
        Selesai
      </label>

      <label>
        <input v-model="isDone" type="radio" :value="false" />
        Belum Selesai
      </label>
    </div>
    <button @click="refetchTodos">refetch</button>
    <div v-if="$apollo.queries.todos.loading">...loading</div>
    <div v-else class="todo-list">
      <div v-for="todo in todos" :key="todo.id" class="todo-item">
        <div :class="['todo']" :title="todo.id">
          <div>
            {{ todo.title }}
          </div>
          <div>
            {{ todo.is_done ? 'selesai' : 'belum selesai' }}
          </div>
        </div>
        <div class="todo-action">
          <div class="todo-delete" @click="deleteTodo(todo.id)">Hapus</div>
          <div class="todo-edit" @click="editTodo(todo)">Edit</div>
        </div>
      </div>
    </div>
    <!-- <div class="todo-list">
      <div v-for="user in users" :key="user.id" class="todo-item">
        <div :class="['todo']" :title="user.id">
          <div>
            {{ user.name }}
          </div>
        </div>
        <div class="todo-delete" @click="deleteTodo(user.id)">Hapus</div>
      </div>
    </div> -->
  </div>
</template>

<style scoped>
body,
input {
  font-family: Helvetica, sans-serif;
  font-size: 12pt;
}

#app {
  max-width: 500px;
  padding: 12px;
  margin: auto;
  text-align: center;
}

.info,
.loading {
  color: #999;
  margin: 12px;
}

form {
  margin: 22px;
}

input {
  padding: 8px;
  border: solid 1px #bbb;
  border-radius: 2px;
}

input:focus {
  box-shadow: none;
  outline: none;
  border-color: #40b883;
}

.todo-list {
  display: flex;
  flex-direction: column;
  text-align: left;
  border: solid 1px #40b883;
  padding: 12px;
  gap: 8px;
  border-radius: 3px;
}

.todo-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border: solid 1px #40b883;
  padding: 8px;
  border-radius: 4px;
}

.todo {
  display: inline-block;
  padding: 4px;
  color: #40b883;
}

.todo.optimistic {
  background: #f7e75e;
}
.todo-action {
  display: flex;
  gap: 4px;
}

.todo-delete {
  padding: 4px;
  cursor: pointer;
  border-radius: 4px;
  color: white;
  background-color: #fa6262;
}
.todo-edit {
  padding: 4px;
  cursor: pointer;
  border-radius: 4px;
  color: white;
  background-color: #5bf125;
}

.actions {
  text-align: center;
}
</style>
