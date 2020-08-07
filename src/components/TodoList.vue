/* eslint no-tabs: ["error", { allowIndentationTabs: true }] */
<template>
  <div>
		<v-card width="550" tile elevation="4">
			<v-text-field
				dense
				solo
				flat
				v-model="newTodo"
				class="text-h5 pl-8 pt-3 pb-3"
				hide-details="auto"
				placeholder="What needs to be done?"
				@keyup.enter="addTodo"
			>
				<v-icon
					v-show="todoList.length"
					slot="prepend"
					class="ml-n6 mt-n1"
					large
					color="grey lighten-1"
					@click="toggleAllTodosChecked"
				>mdi-chevron-down
				</v-icon>
			</v-text-field>

			<v-list
				flat
				v-if="todoList.length"
				class="todos-list py-0"
				id="todos-list"
			>
				<v-list-item-group
					v-for="(todo, ind) in todoList"
					:key="ind"
					v-show="todo.isShow"
				>
					<v-hover v-slot:default="{ hover }">
					<v-list-item class="py-1">
						<div
							@click="toggleChecked(todo)"
							color="grey lighten-1"
							class="check d-flex align-center justify-center mr-7 ml-n2"
						>
							<v-icon v-if="todo.isDone" color="green lighten-2">mdi-check</v-icon>
						</div>
						<v-list-item-content class="py-1">
							<div @dblclick="editTodo(todo, ind)" @blur="endEdit(todo)">
								<v-text-field
									solo
									flat
									hide-details="auto"
									class="text-h5"
									:class="[todo.isDone && !todo.isEdit ? 'done-todo' : '']"
									type="text"
									@blur="endEdit(todo)"
									:disabled="!todo.isEdit"
									ref="inputTodo"
									v-model="todo.name"
								></v-text-field>
							</div>
						</v-list-item-content>
						<v-list-item-icon :class="{ 'show-close-button': hover }" class="close-button my-auto" @click="removeTodo(todo, ind)">
							<v-icon color="brown" class="text--lighten-3" style="font-size: 1.3em">mdi-close-thick</v-icon>
						</v-list-item-icon>
					</v-list-item>
					</v-hover>
					<v-divider
            v-if="ind + 1 < todoList.length"
            :key="ind"
          ></v-divider>
				</v-list-item-group>
			</v-list>
			<div v-if="todoList.length" class="footer d-flex justify-space-between align-center py-1 px-3">
				<div>
					<span v-if="itemsLeftCount > 1 || !itemsLeftCount" class="grey--text text--darken-2 text-body-2">{{ itemsLeftCount }} items left</span>
					<span v-else class="grey--text text--darken-2 text-body-2">{{ itemsLeftCount }} item left</span>
				</div>
				<div class="pa-1">
					<a
						@click="showAll"
						:class="[isShowAll ? 'active' : '']"
						class="sort-button rounded grey--text text--darken-2 text-body-2 mx-3 py-1 px-2"
					>All</a>
					<a
						@click="showActive"
						:class="[isShowActive ? 'active' : '']"
						class="sort-button rounded grey--text text--darken-2 text-body-2 mx-3 py-1 px-2"
					>Active</a>
					<a
						@click="showCompleted"
						:class="[isShowCompleted ? 'active' : '']"
						class="sort-button rounded grey--text text--darken-2 text-body-2 mx-3 py-1 px-2"
					>Completed</a>
				</div>
				<div class="d-flex align-center">
					<a @click="clearCompleted" class="clear-button grey--text text--darken-2 text-body-2">Clear completed</a>
				</div>
			</div>
		</v-card>
	</div>
</template>

<script>
export default {
  data: () => ({
    newTodo: '',
    todoList: [],
    allTodosChecked: false,
    isShowAll: false,
    isShowActive: false,
    isShowCompleted: false,
    disableInput: true,
    disabled: false
  }),
  computed: {
    itemsLeftCount: function () {
      let itemsLeft = 0
      this.todoList.forEach(el => {
        if (!el.isDone) itemsLeft += 1
      })
      return itemsLeft
    }
  },
  mounted () {
    if (localStorage.getItem('todos')) {
      try {
        this.todoList = JSON.parse(localStorage.getItem('todos'))
      } catch (e) {
        localStorage.removeTodo('todos')
      }
    }
  },
  methods: {
    addTodo () {
      if (!this.newTodo) return
      this.todoList.push({
        name: this.newTodo.trim(),
        isDone: false,
        isShow: true,
        isEdit: false
      })
      this.newTodo = ''
      this.saveTodo()
    },
    saveTodo () {
      const parsed = JSON.stringify(this.todoList)
      localStorage.setItem('todos', parsed)
    },
    removeTodo (todo) {
      this.todoList.splice(todo, 1)
      this.saveTodo()
    },
    toggleChecked (todo, ind) {
      todo.isDone = !todo.isDone
      this.saveTodo()
    },
    toggleAllTodosChecked () {
      const isSomeTodoChecked = this.todoList.some(el => el.isDone)

      switch (isSomeTodoChecked) {
        case (isSomeTodoChecked && !this.allTodosChecked):
          this.todoList.forEach(el => (el.isDone = true))
          this.allTodosChecked = true
          break
        case (this.allTodosChecked): {
          this.todoList.forEach(el => (el.isDone = false))
          this.allTodosChecked = false
          break
        }
      }
      this.saveTodo()
    },
    clearCompleted () {
      this.todoList = this.todoList.filter(el => !el.isDone)
      this.saveTodo()
    },
    showActive () {
      this.isShowActive = true
      this.isShowCompleted = false
      this.isShowAll = false
      this.todoList.forEach(el => {
        if (el.isDone) return (el.isShow = false)
        el.isShow = true
      })
    },
    showCompleted () {
      this.isShowActive = false
      this.isShowCompleted = true
      this.isShowAll = false
      this.todoList.forEach(el => {
        if (el.isDone) return (el.isShow = true)
        el.isShow = false
      })
    },
    showAll () {
      this.isShowActive = false
      this.isShowCompleted = false
      this.isShowAll = true
      this.todoList.forEach(el => (el.isShow = true))
    },
    editTodo (todo, ind) {
      todo.isEdit = true
      setTimeout(() => (this.$refs.inputTodo[ind].focus()), 1)
    },
    endEdit (todo) {
      todo.isEdit = false
      this.saveTodo()
    }
  }
}
</script>

<style scoped>
	#todos-list {
		border-top: 1px rgba(0, 0, 0, 0.12) solid
	}

	.footer {
		border-top: 1px rgba(0, 0, 0, 0.12) solid
	}

	.clear-button:hover {
		text-decoration: underline;
	}

	.sort-button:hover {
		border: 1px rgba(175, 47, 47, 0.1) solid;
	}

	.sort-button {
		border: 1px transparent solid;
	}

	.active {
		border: 1px rgba(175, 47, 47, 0.2) solid;
	}

	.close-button {
		display: none;
	}

	.show-close-button {
		display: inline;
	}

	.check {
		border-radius: 50%;
		width: 2em;
		height: 2em;
		border: rgb(195, 195, 195) 1px solid
	}

	.done-todo {
		text-decoration: line-through;
		color: rgb(195, 195, 195);
	}
</style>
