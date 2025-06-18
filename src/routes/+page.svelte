<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';

  const notes = writable([]);
  const apiURL = 'https://YOUR_MOCKAPI_URL/notes'; // replace with your mockapi url

  let title = '';
  let content = '';
  let editingId = null;

  async function fetchNotes() {
    const res = await fetch(apiURL);
    notes.set(await res.json());
  }

  async function addNote() {
    if (!title || !content) return;
    await fetch(apiURL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title, content, createdAt: new Date() })
    });
    title = content = '';
    fetchNotes();
  }

  async function deleteNote(id) {
    await fetch(`${apiURL}/${id}`, { method: 'DELETE' });
    fetchNotes();
  }

  function startEdit(note) {
    title = note.title;
    content = note.content;
    editingId = note.id;
  }

  async function saveEdit() {
    await fetch(`${apiURL}/${editingId}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title, content, createdAt: new Date() })
    });
    title = content = '';
    editingId = null;
    fetchNotes();
  }

  onMount(fetchNotes);
</script>

<div class="max-w-xl mx-auto p-4 space-y-4">
  <h1 class="text-2xl font-bold">Notes App</h1>

  <form class="space-y-2" on:submit|preventDefault={editingId ? saveEdit : addNote}>
    <input bind:value={title} type="text" placeholder="Title" class="w-full p-2 border rounded" required />
    <textarea bind:value={content} placeholder="Content" class="w-full p-2 border rounded" required></textarea>
    <button type="submit" class="px-4 py-2 bg-blue-600 text-white rounded">{editingId ? 'Save Note' : 'Add Note'}</button>
  </form>

  <ul class="space-y-2">
    {#each $notes as note}
      <li class="border p-2 rounded shadow-sm">
        <h2 class="font-semibold">{note.title}</h2>
        <p class="text-sm">{note.content}</p>
        <div class="mt-2 space-x-2">
          <button class="text-blue-600" on:click={() => startEdit(note)}>Edit</button>
          <button class="text-red-600" on:click={() => deleteNote(note.id)}>Delete</button>
        </div>
      </li>
    {/each}
  </ul>
</div>

<style>
  :global(body) {
    font-family: sans-serif;
  }
</style>
