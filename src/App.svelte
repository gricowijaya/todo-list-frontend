<script>
  import { afterUpdate, onMount } from "svelte";
  import axios from 'axios';

  let todoItems = [];
  let newTodo = "";
  let newDescription = "";

  onMount(async function getTodo() {
    try {
      const response = await axios.get(`${API_HOST}/read/all`);
      response.data.forEach((todo) => {todoItems.push(todo)})
    } catch (err) {
      console.error(err);
    }
  });

  async function addTodo() {
    newTodo = newTodo.trim();
    newDescription = newDescription.trim();
    if (!newTodo) return;

    const todo = {
      name: newTodo,
      description: newDescription,
    };

    // post into server
    await axios.post(`${API_HOST}/create`, todo);
    todoItems = [...todoItems, todo] // put the data into the html
    newTodo = "";
    newDescription = "";
  }

  async function toggleDone(id) {
    const index = todoItems.findIndex((item) => item.id === Number(id));
    await axios.put(`${API_HOST}/done`, { id: id });
    todoItems[index].is_done = !todoItems[index].is_done;
  }

  async function deleteTodo(id) {
    await axios.delete(`${API_HOST}/delete`, { data: { id: id } });
    todoItems = todoItems.filter((item) => item.id !== Number(id));
  }
</script>

<main>
  <div class="container">
    <h1 class="app-title">todos</h1>

    <ul class="todo-list">
      {#each todoItems as todo (todo.id)}
      <li class="todo-item {todo.is_done ? 'done' : ''}">
        <input id={todo.id} type="checkbox" />
        <label
          for={todo.id}
          class="tick"
          on:click={() => toggleDone(todo.id)}
        />
        <span>{todo.name}</span>
        <button class="delete-todo" on:click={() => deleteTodo(todo.id)}>
          <svg><use href="#delete-icon" /></svg>
        </button>
      </li>
      {/each}
    </ul>
    <form on:submit|preventDefault={addTodo}>
      <input
        class="js-todo-input"
        type="text"
        aria-label="Enter a new todo item"
        placeholder="E.g. Build a web app"
        bind:value={newTodo}
      />
    </form>
    <form on:submit|preventDefault={addTodo}>
      <input
        class="js-todo-input-description"
        type="text"
        aria-label="Enter the description"
        placeholder="Create the description for your todo!"
        bind:value={newDescription}
      />
    </form>
  </div>
</main>
