
# Svelte 4 vs Svelte 5 - Complete Comparison Reference

## Reactivity

### Declare state

**Svelte 4:**
```svelte
<script>
  let name = "John";
</script>

<h1>Hello {name}</h1>
```

**Svelte 5:**
```svelte
<script>
  let name = $state("John");
</script>

<h1>Hello {name}</h1>
```

### Update state

**Svelte 4:**
```svelte
<script>
  let name = "John";
  name = "Jane";
</script>

<h1>Hello {name}</h1>
```

**Svelte 5:**
```svelte
<script>
  let name = $state("John");
  name = "Jane";
</script>

<h1>Hello {name}</h1>
```

### Computed state

**Svelte 4:**
```svelte
<script>
  let count = 10;
  $: doubleCount = count * 2;
</script>

<div>{doubleCount}</div>
```

**Svelte 5:**
```svelte
<script>
  let count = $state(10);
  const doubleCount = $derived(count * 2);
</script>

<div>{doubleCount}</div>
```

---

## Templating

### Minimal template

**Svelte 4:**
```svelte
<h1>Hello world</h1>
```

**Svelte 5:**
```svelte
<h1>Hello world</h1>
```

### Styling

**Svelte 4:**
```svelte
<h1 class="title">I am red</h1>
<button style="font-size: 10rem;">I am a button</button>

<style>
  .title {
    color: red;
  }
</style>
```

**Svelte 5:**
```svelte
<h1 class="title">I am red</h1>
<button style="font-size: 10rem;">I am a button</button>

<style>
  .title {
    color: red;
  }
</style>
```

### Loop

**Svelte 4:**
```svelte
<script>
  const colors = ["red", "green", "blue"];
</script>

<ul>
  {#each colors as color (color)}
    <li>{color}</li>
  {/each}
</ul>
```

**Svelte 5:**
```svelte
<script>
  const colors = ["red", "green", "blue"];
</script>

<ul>
  {#each colors as color (color)}
    <li>{color}</li>
  {/each}
</ul>
```

### Event click

**Svelte 4:**
```svelte
<script>
  let count = 0;
  
  function incrementCount() {
    count++;
  }
</script>

<p>Counter: {count}</p>
<button on:click={incrementCount}>+1</button>
```

**Svelte 5:**
```svelte
<script>
  let count = $state(0);
  
  function incrementCount() {
    count++;
  }
</script>

<p>Counter: {count}</p>
<button onclick={incrementCount}>+1</button>
```

### Dom ref

**Svelte 4:**
```svelte
<script>
  import { onMount } from "svelte";
  
  let inputElement;
  
  onMount(() => {
    inputElement.focus();
  });
</script>

<input bind:this={inputElement} />
```

**Svelte 5:**
```svelte
<script>
  let inputElement;
  
  $effect(() => {
    inputElement.focus();
  });
</script>

<input bind:this={inputElement} />
```

### Conditional

**Svelte 4:**
```svelte
<script>
  const TRAFFIC_LIGHTS = ["red", "orange", "green"];
  let lightIndex = 0;
  $: light = TRAFFIC_LIGHTS[lightIndex];
  
  function nextLight() {
    lightIndex = (lightIndex + 1) % TRAFFIC_LIGHTS.length;
  }
</script>

<button on:click={nextLight}>Next light</button>
<p>Light is: {light}</p>
<p>
  You must
  {#if light === "red"}
    <span>STOP</span>
  {:else if light === "orange"}
    <span>SLOW DOWN</span>
  {:else if light === "green"}
    <span>GO</span>
  {/if}
</p>
```

**Svelte 5:**
```svelte
<script>
  const TRAFFIC_LIGHTS = ["red", "orange", "green"];
  let lightIndex = $state(0);
  const light = $derived(TRAFFIC_LIGHTS[lightIndex]);
  
  function nextLight() {
    lightIndex = (lightIndex + 1) % TRAFFIC_LIGHTS.length;
  }
</script>

<button onclick={nextLight}>Next light</button>
<p>Light is: {light}</p>
<p>
  You must
  {#if light === "red"}
    <span>STOP</span>
  {:else if light === "orange"}
    <span>SLOW DOWN</span>
  {:else if light === "green"}
    <span>GO</span>
  {/if}
</p>
```

---

## Lifecycle

### On mount

**Svelte 4:**
```svelte
<script>
  import { onMount } from "svelte";
  
  let pageTitle = "";
  
  onMount(() => {
    pageTitle = document.title;
  });
</script>

<p>Page title: {pageTitle}</p>
```

**Svelte 5:**
```svelte
<script>
  let pageTitle = $state("");
  
  $effect(() => {
    pageTitle = document.title;
  });
</script>

<p>Page title: {pageTitle}</p>
```

### On unmount

**Svelte 4:**
```svelte
<script>
  import { onDestroy } from "svelte";
  
  let time = new Date().toLocaleTimeString();
  
  const timer = setInterval(() => {
    time = new Date().toLocaleTimeString();
  }, 1000);
  
  onDestroy(() => clearInterval(timer));
</script>

<p>Current time: {time}</p>
```

**Svelte 5:**
```svelte
<script>
  let time = $state(new Date().toLocaleTimeString());
  
  $effect(() => {
    const timer = setInterval(() => {
      time = new Date().toLocaleTimeString();
    }, 1000);
    
    return () => clearInterval(timer);
  });
</script>

<p>Current time: {time}</p>
```

---

## Component composition

### Props

**Svelte 4:**
```svelte
<script>
  import UserProfile from "./UserProfile.svelte";
</script>

<UserProfile
  name="John"
  age={20}
  favouriteColors={["green", "blue", "red"]}
  isAvailable
/>
```

**Svelte 5:**
```svelte
<script>
  import UserProfile from "./UserProfile.svelte";
</script>

<UserProfile
  name="John"
  age={20}
  favouriteColors={["green", "blue", "red"]}
  isAvailable
/>
```

### Emit to parent

**Svelte 4:**
```svelte
<script>
  import AnswerButton from "./AnswerButton.svelte";
  
  let isHappy = true;
  
  function onAnswerNo() {
    isHappy = false;
  }
  
  function onAnswerYes() {
    isHappy = true;
  }
</script>

<p>Are you happy?</p>
<AnswerButton on:yes={onAnswerYes} on:no={onAnswerNo} />
<p style="font-size: 50px;">{isHappy ? "😀" : "😥"}</p>
```

**Svelte 5:**
```svelte
<script>
  import AnswerButton from "./AnswerButton.svelte";
  
  let isHappy = $state(true);
  
  function onAnswerNo() {
    isHappy = false;
  }
  
  function onAnswerYes() {
    isHappy = true;
  }
</script>

<p>Are you happy?</p>
<AnswerButton onYes={onAnswerYes} onNo={onAnswerNo} />
<p style="font-size: 50px;">{isHappy ? "😀" : "😥"}</p>
```

### Slot

**Svelte 4:**
```svelte
<script>
  import FunnyButton from "./FunnyButton.svelte";
</script>

<FunnyButton>Click me!</FunnyButton>
```

**Svelte 5:**
```svelte
<script>
  import FunnyButton from "./FunnyButton.svelte";
</script>

<FunnyButton>Click me!</FunnyButton>
```

### Slot fallback

**Svelte 4:**
```svelte
<script>
  import FunnyButton from "./FunnyButton.svelte";
</script>

<FunnyButton />
<FunnyButton>I got content!</FunnyButton>
```

**Svelte 5:**
```svelte
<script>
  import FunnyButton from "./FunnyButton.svelte";
</script>

<FunnyButton />
<FunnyButton>I got content!</FunnyButton>
```

### Context

**Svelte 4:**
```svelte
<script>
  import { setContext } from "svelte";
  import UserProfile from "./UserProfile.svelte";
  import createUserStore from "./createUserStore.js";
  
  const userStore = createUserStore({
    id: 1,
    username: "unicorn42",
    email: "unicorn42@example.com",
  });
  
  setContext("user", userStore);
</script>

<h1>Welcome back, {$userStore.username}</h1>
<UserProfile />
```

**Svelte 5:**
```svelte
<script>
  import { setContext } from "svelte";
  import UserProfile from "./UserProfile.svelte";
  import createUserState from "./createUserState.svelte.js";
  
  const user = createUserState({
    id: 1,
    username: "unicorn42",
    email: "unicorn42@example.com",
  });
  
  setContext("user", user);
</script>

<h1>Welcome back, {user.username}</h1>
<UserProfile />
```

---

## Form input

### Input text

**Svelte 4:**
```svelte
<script>
  let text = "Hello World";
</script>

<p>{text}</p>
<input bind:value={text} />
```

**Svelte 5:**
```svelte
<script>
  let text = $state("Hello World");
</script>

<p>{text}</p>
<input bind:value={text} />
```

### Checkbox

**Svelte 4:**
```svelte
<script>
  let isAvailable = false;
</script>

<input id="is-available" type="checkbox" bind:checked={isAvailable} />
<label for="is-available">Is available</label>
```

**Svelte 5:**
```svelte
<script>
  let isAvailable = $state(false);
</script>

<input id="is-available" type="checkbox" bind:checked={isAvailable} />
<label for="is-available">Is available</label>
```

### Select

**Svelte 4:**
```svelte
<script>
  let selectedColorId = 2;
  
  const colors = [
    { id: 1, text: "red" },
    { id: 2, text: "blue" },
    { id: 3, text: "green" },
    { id: 4, text: "gray", isDisabled: true },
  ];
</script>

<select bind:value={selectedColorId}>
  {#each colors as color}
    <option value={color.id} disabled={color.isDisabled}>
      {color.text}
    </option>
  {/each}
</select>
```

**Svelte 5:**
```svelte
<script>
  let selectedColorId = $state(2);
  
  const colors = [
    { id: 1, text: "red" },
    { id: 2, text: "blue" },
    { id: 3, text: "green" },
    { id: 4, text: "gray", isDisabled: true },
  ];
</script>

<select bind:value={selectedColorId}>
  {#each colors as color}
    <option value={color.id} disabled={color.isDisabled}>
      {color.text}
    </option>
  {/each}
</select>
```

---

## Webapp features

### Render app

**Svelte 4:**
```html
<!doctype html>
<html>
  <body>
    <div id="app"></div>
    <script type="module" src="./app.js"></script>
  </body>
</html>
```

**Svelte 5:**
```html
<!doctype html>
<html>
  <body>
    <div id="app"></div>
    <script type="module" src="./app.js"></script>
  </body>
</html>
```

### Fetch data

**Svelte 4:**
```svelte
<script>
  import useFetchUsers from "./useFetchUsers";
  
  const { isLoading, error, data: users } = useFetchUsers();
</script>

{#if $isLoading}
  <p>Fetching users...</p>
{:else if $error}
  <p>An error occurred while fetching users</p>
{:else if $users}
  <ul>
    {#each $users as user}
      <li>
        <img src={user.picture.thumbnail} alt="user" />
        <p>
          {user.name.first}
          {user.name.last}
        </p>
      </li>
    {/each}
  </ul>
{/if}
```

**Svelte 5:**
```svelte
<script>
  import useFetchUsers from "./useFetchUsers.svelte.js";
  
  const response = useFetchUsers();
</script>

{#if response.isLoading}
  <p>Fetching users...</p>
{:else if response.error}
  <p>An error occurred while fetching users</p>
{:else if response.users}
  <ul>
    {#each response.users as user}
      <li>
        <img src={user.picture.thumbnail} alt="user" />
        <p>
          {user.name.first}
          {user.name.last}
        </p>
      </li>
    {/each}
  </ul>
{/if}
```

---

## Key Differences Summary

### Migration Lessons from Svelte 5 Runes
- **Props extraction**: Call `$props()` without generics, then apply TypeScript annotations on the destructured target. Example:
  ```svelte
  interface ButtonProps { variant?: 'primary' | 'secondary'; }
  let { variant = 'primary' }: ButtonProps = $props();
  ```
- **Derived state**: Pass expressions directly into `$derived` (avoid wrapping in arrow functions) so the rune can track dependencies correctly.
  ```svelte
  const classes = $derived([base, variantClasses[variant]].join(' '));
  ```
- **Slots as snippets**: Prefer Svelte 5’s `Snippet` props instead of `$slots()`. Declare slot props on the component’s interface and render them conditionally.
  ```svelte
  import type { Snippet } from 'svelte';

  interface CardProps { header?: Snippet; children?: Snippet; footer?: Snippet; }
  let { header, children, footer }: CardProps = $props();

  {#if header}
    {@render header()}
  {/if}
  ```
- **Legacy slot helpers**: If you must inspect `$slots()`, declare a typed variable in two steps (`let slots: Slots; slots = $slots();`) to avoid “implicit any” and declaration-order diagnostics. However, prefer the `Snippet` pattern when possible.
- **Validation workflow**: After migrating primitives, run targeted `svelte-check` commands (e.g. `npx svelte-check src/lib/ui/primitives/Button.svelte`) to confirm the new rune patterns compile cleanly before running project-wide checks.

### Reactivity
- **Svelte 4**: Implicit reactivity with `let` declarations and `$:` reactive statements
- **Svelte 5**: Explicit reactivity using runes: `$state()` for state, `$derived()` for computed values

### Event Handling
- **Svelte 4**: `on:click={handler}` directive syntax
- **Svelte 5**: `onclick={handler}` native DOM attribute syntax

### Lifecycle
- **Svelte 4**: Separate `onMount()` and `onDestroy()` functions
- **Svelte 5**: Unified `$effect()` rune with cleanup via return function

### Component Events
- **Svelte 4**: Event dispatching with `on:eventName={handler}`
- **Svelte 5**: Callback props pattern with `onEventName={handler}`

### Context and Stores
- **Svelte 4**: Uses stores (with `$` prefix for auto-subscription)
- **Svelte 5**: Uses state objects directly (no `$` prefix needed for reactivity)

### DOM References
- **Svelte 4**: Requires `onMount()` import for DOM manipulation
- **Svelte 5**: Uses `$effect()` rune (no separate import needed)
