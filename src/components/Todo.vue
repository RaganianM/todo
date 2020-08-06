<template>
	<v-app>
		<v-main class="d-flex flex-column align-center grey lighten-4">
			<div class="d-flex">
				<span class="header text-h1 mx-auto font-weight-regular my-4">todos</span>
			</div>
			<TodoList />
			<div class="footer d-flex">
				<span class="caption mx-auto mt-14 grey--text">Double-click to edit a todo</span>
			</div>
		</v-main>
	</v-app>
</template>

<script>
import TodoList from './TodoList'

export default {
  data: () => ({}),
  components: {
    TodoList
  }
}
</script>

<style scoped>
	.header {
		color: rgba(175, 47, 47, 0.15);
	}

	.caption {
		font-size: 0.6em
	}
</style>
