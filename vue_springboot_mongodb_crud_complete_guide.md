# Vue 3 + Spring Boot + MongoDB CRUD Frontend --- Complete Guide

A detailed, practical guide to building a Vue 3 frontend for a Spring
Boot REST API backed by MongoDB. It explains project files, HTML forms
and tables, the Composition API, `ref`, `reactive`, arrow functions,
`computed`, `watch`, props and emits, Vue Router and child routes, Pinia
Setup Stores, Axios, and the complete CRUD flow.

> **Approach used in this guide**
>
> -   Vue 3 Composition API with `<script setup>`
> -   Vue Router
> -   **Pinia Setup Store** using `ref()`, `computed()` and functions
> -   Axios calls directly inside the Pinia store
> -   **No separate `services` folder**
> -   Spring Boot provides REST endpoints; MongoDB is accessed by Spring
>     Boot, never directly by Vue

------------------------------------------------------------------------

## 1. Understand the complete architecture

The application has three main parts:

``` text
Vue 3 frontend (browser)
  ├── Views/pages
  ├── Reusable components
  ├── Vue Router
  └── Pinia store
        └── Axios HTTP requests
              ↓ JSON over HTTP
Spring Boot REST API
  ├── Controller
  ├── Service (if your backend uses one)
  └── Repository
        ↓
MongoDB database
```

For example, when a user creates a student:

1.  The user fills in a Vue form.
2.  `v-model` keeps the form data updated.
3.  The component calls a Pinia action such as `addStudent(form)`.
4.  The Pinia action sends an Axios `POST` request to Spring Boot.
5.  Spring Boot validates and saves the student through its repository.
6.  Spring Boot returns a response, usually JSON.
7.  Pinia updates its local state.
8.  Vue reactively updates the page.

**Important:** Vue does not connect directly to MongoDB. The frontend
talks to the Spring Boot API.

### Example backend endpoints

The examples below assume the backend exposes these endpoints. Adjust
them to match your controller.

  Operation           HTTP method   Example endpoint
  ------------------- ------------- ----------------------
  Read all students   `GET`         `/api/students`
  Read one student    `GET`         `/api/students/{id}`
  Create student      `POST`        `/api/students`
  Update student      `PUT`         `/api/students/{id}`
  Delete student      `DELETE`      `/api/students/{id}`

The examples assume a student has `id`, `name`, `email` and `department`
properties. If your API returns MongoDB's `_id` rather than `id`,
consistently use the field your backend actually returns.

------------------------------------------------------------------------

## 2. Create the Vue project

Install Node.js, then run:

``` bash
npm create vue@latest
```

A typical set of choices for this guide is:

-   Project name: `student-frontend`
-   TypeScript: No
-   JSX: No
-   Vue Router: Yes
-   Pinia: Yes
-   Tests: optional
-   ESLint / Prettier: optional

Then:

``` bash
cd student-frontend
npm install
npm install axios
npm run dev
```

The development server commonly runs at `http://localhost:5173`. Spring
Boot commonly runs at `http://localhost:8080`.

-   `npm install` installs dependencies.
-   `npm install axios` adds the HTTP client.
-   `npm run dev` starts the frontend development server.

If Router or Pinia were not selected during project setup, install them:

``` bash
npm install vue-router pinia
```

------------------------------------------------------------------------

## 3. Understand the project files

A useful structure is:

``` text
student-frontend/
├── index.html
├── package.json
└── src/
    ├── main.js
    ├── App.vue
    ├── router/
    │   └── index.js
    ├── stores/
    │   └── studentStore.js
    ├── views/
    │   ├── StudentLayout.vue
    │   ├── StudentList.vue
    │   ├── StudentCreate.vue
    │   └── StudentDetails.vue
    ├── components/
    │   └── StudentForm.vue
    └── assets/
        └── main.css
```

  -----------------------------------------------------------------------
  File/folder                         Purpose
  ----------------------------------- -----------------------------------
  `index.html`                        The HTML page containing Vue's
                                      mounting element.

  `main.js`                           Creates the Vue app and registers
                                      Pinia, Router and global CSS.

  `App.vue`                           Root component. Usually contains
                                      the top-level `<RouterView />`.

  `router/index.js`                   Defines URL paths and which
                                      component each route displays.

  `stores/studentStore.js`            Shared student state, computed
                                      values and CRUD actions. Axios is
                                      called directly here.

  `views/`                            Full pages matched to routes.

  `components/`                       Reusable UI pieces, such as a form
                                      used for both create and edit.

  `assets/main.css`                   Global styling.

  `package.json`                      Project metadata, dependencies and
                                      npm scripts.
  -----------------------------------------------------------------------

Create any missing folders inside `src`.

------------------------------------------------------------------------

## 4. The entry point: `main.js`

``` javascript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'
import './assets/main.css'

const app = createApp(App)

app.use(createPinia())
app.use(router)

app.mount('#app')
```

Explanation:

-   `createApp(App)` creates the Vue application with `App.vue` as its
    root.
-   `createPinia()` creates the Pinia instance.
-   `app.use(createPinia())` makes the store available throughout the
    application.
-   `app.use(router)` enables Vue Router.
-   The CSS import loads global styles.
-   `app.mount('#app')` mounts Vue into the element with ID `app` in
    `index.html`.

A typical `index.html` contains:

``` html
<div id="app"></div>
<script type="module" src="/src/main.js"></script>
```

Vue renders the application inside `#app`.

------------------------------------------------------------------------

## 5. Vue Single File Components

A Vue component commonly contains three sections:

``` vue
<template>
  <h1>{{ title }}</h1>
</template>

<script setup>
const title = 'Student Management'
</script>

<style scoped>
h1 {
  color: blue;
}
</style>
```

-   `<template>` contains the markup to render.
-   `<script setup>` contains JavaScript: variables, imports, state and
    functions.
-   `<style scoped>` contains component-specific CSS. `scoped` prevents
    these styles from automatically applying to elements in other
    components.

With `<script setup>`, top-level variables, functions and imported
components can be used directly in the template. You do not need the
older Options API structure for these examples.

### Root component: `App.vue`

``` vue
<template>
  <RouterView />
</template>
```

`<RouterView />` displays the page component matched by the current URL.

------------------------------------------------------------------------

## 6. Arrow functions: `() => {}`

An arrow function is a shorter way to write a JavaScript function.

### Normal function

``` javascript
function add(a, b) {
  return a + b
}
```

### Equivalent arrow function

``` javascript
const add = (a, b) => {
  return a + b
}
```

-   `(a, b)` are the parameters.
-   `=>` separates parameters from the function body.
-   `{ ... }` contains the statements.
-   `return` sends a value back to the caller.

### Common forms

``` javascript
// No parameters
const greet = () => {
  console.log('Hello')
}

// One parameter: parentheses are optional
const double = n => {
  return n * 2
}

// One-line expression: returned automatically
const square = n => n * n

// Multiple parameters
const multiply = (a, b) => a * b
```

If an arrow function uses braces, it does not automatically return a
value:

``` javascript
const a = n => n * 2
const b = n => { n * 2 }          // returns undefined
const c = n => { return n * 2 }   // returns the result
```

### Why does Pinia use `state: () => ({ ... })`?

``` javascript
state: () => ({
  students: [],
  loading: false
})
```

The function takes no parameters and returns an object. Parentheses
around the object are necessary for the concise expression-body syntax;
otherwise `{}` would be interpreted as a function body.

Equivalent version:

``` javascript
state: () => {
  return {
    students: [],
    loading: false
  }
}
```

A state factory returns an initial state object. In Pinia's Setup Store
syntax used later in this guide, you instead declare state using `ref()`
and return it.

### Arrow functions and `this`

In a Pinia **Options Store**, actions commonly use method syntax so
`this` refers to the store:

``` javascript
actions: {
  clearStudents() {
    this.students = []
  }
}
```

Avoid using an arrow function for an Options Store action that needs the
store's `this`, because arrow functions inherit `this` from their
surrounding scope.

This guide uses a **Setup Store**, where state is referenced by variable
names such as `students.value`, so this Options Store rule is not needed
for store actions.

------------------------------------------------------------------------

## 7. `ref()` and `reactive()`

These create reactive state. When reactive values change, Vue updates
the relevant rendered UI.

### 7.1 `ref()`

Use `ref()` for primitive values, objects and arrays.

``` vue
<script setup>
import { ref } from 'vue'

const name = ref('')
const age = ref(20)
const loading = ref(false)
const students = ref([])
</script>

<template>
  <p>{{ name }}</p>
  <p>{{ age }}</p>
</template>
```

In JavaScript, read or update a ref using `.value`:

``` javascript
name.value = 'Arun'
age.value = 21
students.value = [{ id: '1', name: 'Arun' }]
```

In templates, Vue automatically unwraps refs:

``` html
<p>{{ name }}</p>
```

You normally do not write `name.value` in the template.

### 7.2 `reactive()`

Use `reactive()` for an object whose properties you want to access
directly.

``` javascript
import { reactive } from 'vue'

const form = reactive({
  name: '',
  email: '',
  department: ''
})

form.name = 'Arun'
form.email = 'arun@example.com'
```

There is no `.value` when accessing properties on this reactive object.

### `ref()` vs `reactive()`

  -----------------------------------------------------------------------
  Feature                 `ref()`                 `reactive()`
  ----------------------- ----------------------- -----------------------
  Strings, numbers,       Yes                     Not directly
  booleans                                        

  Objects and arrays      Yes                     Yes

  Access in JavaScript    `.value`                Direct property

  Replace whole value     Assign to `.value`      Usually mutate existing
                                                  properties; do not
                                                  replace the proxy
                                                  variable itself

  Typical use             Loading flags, IDs,     Form objects with
                          arrays, selected record related fields
  -----------------------------------------------------------------------

Both are valid for forms and arrays. Pick the style that makes your code
clear.

### 7.3 `v-model`

`v-model` connects an input to reactive state.

``` vue
<script setup>
import { ref } from 'vue'

const name = ref('')
</script>

<template>
  <input v-model="name" placeholder="Enter name">
  <p>Name: {{ name }}</p>
</template>
```

Typing changes `name`, and changing `name` in JavaScript updates the
input.

For a reactive object:

``` html
<input v-model="form.name">
<input v-model="form.email">

<select v-model="form.department">
  <option value="">Select department</option>
  <option value="CSE">CSE</option>
  <option value="ECE">ECE</option>
</select>
```

`v-model` is particularly useful for create and edit forms.

------------------------------------------------------------------------

## 8. Important Vue directives

  ------------------------------------------------------------------------------------------
  Directive               Purpose                 Example
  ----------------------- ----------------------- ------------------------------------------
  `v-if`                  Conditionally           `<p v-if="loading">Loading...</p>`
                          creates/removes an      
                          element                 

  `v-else`                Alternative branch      `<p v-else>Finished</p>`

  `v-show`                Toggles CSS display     `<p v-show="visible">Hello</p>`
                          while keeping element   
                          in DOM                  

  `v-for`                 Repeats markup for an   `<li v-for="s in students" :key="s.id">`
                          array                   

  `v-bind` / `:`          Binds an attribute to   `<input :disabled="loading">`
                          an expression           

  `v-on` / `@`            Listens for an event    `<button @click="save">Save</button>`

  `v-model`               Synchronizes input and  `<input v-model="form.name">`
                          state                   
  ------------------------------------------------------------------------------------------

Use a unique and stable `:key` for list rows. For CRUD tables, the
backend ID is usually a good choice.

### Event modifiers

``` html
<form @submit.prevent="saveStudent">
```

`.prevent` prevents the browser's default form submission, which would
otherwise typically reload the page. Vue then calls `saveStudent()`.

------------------------------------------------------------------------

## 9. Basic HTML for CRUD

A typical CRUD page needs forms, inputs, dropdowns, buttons and tables.

### 9.1 Form

``` html
<form @submit.prevent="saveStudent">
  <label>Name</label>
  <input v-model="form.name" required>

  <label>Email</label>
  <input v-model="form.email" type="email" required>

  <button type="submit">Save</button>
</form>
```

-   `<form>` groups related fields.
-   `<label>` describes an input.
-   `<input>` collects text or other values.
-   `type="email"` enables basic email-format validation by the browser.
-   `required` prevents submission while the field is empty.
-   `type="submit"` submits the form, triggering its submit event.

Browser validation is useful but does not replace server-side
validation.

### 9.2 Dropdown

``` html
<select v-model="form.department" required>
  <option value="">Select department</option>
  <option value="CSE">CSE</option>
  <option value="ECE">ECE</option>
  <option value="IT">IT</option>
</select>
```

The selected option is stored in `form.department`.

### 9.3 Table

``` html
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th>Actions</th>
    </tr>
  </thead>

  <tbody>
    <tr v-for="s in students" :key="s.id">
      <td>{{ s.name }}</td>
      <td>{{ s.email }}</td>
      <td>
        <button @click="editStudent(s)">Edit</button>
        <button @click="deleteStudent(s.id)">Delete</button>
      </td>
    </tr>
  </tbody>
</table>
```

-   `<table>` creates the table.
-   `<thead>` groups column headings.
-   `<tbody>` groups data rows.
-   `<tr>` is a row.
-   `<th>` is a heading cell.
-   `<td>` is a data cell.
-   `v-for` repeats the row for each student.
-   `:key="s.id"` helps Vue track each record.

------------------------------------------------------------------------

## 10. Components, props and emits

A component is a reusable piece of UI. For example, a `StudentForm.vue`
can be used for both creation and editing.

### Data flow

``` text
Parent component
   |
   | props (data goes down)
   v
Child component
   |
   | emit (event and optional data go up)
   v
Parent component handles the event
```

### 10.1 `defineProps()` --- parent to child

`components/StudentCard.vue`:

``` vue
<script setup>
defineProps({
  name: String,
  department: String
})
</script>

<template>
  <div>
    <h3>{{ name }}</h3>
    <p>{{ department }}</p>
  </div>
</template>
```

Parent:

``` vue
<script setup>
import StudentCard from '../components/StudentCard.vue'
</script>

<template>
  <StudentCard name="Arun" department="CSE" />
</template>
```

The parent passes `name` and `department` as props. `defineProps()`
declares the data the child accepts. Props are read-only from the
child's perspective; do not directly mutate a prop to change the
parent's data.

### 10.2 `defineEmits()` --- child to parent

Child component:

``` vue
<script setup>
import { ref } from 'vue'

const name = ref('')
const emit = defineEmits(['save'])

function submitForm() {
  emit('save', name.value)
}
</script>

<template>
  <form @submit.prevent="submitForm">
    <input v-model="name" required>
    <button type="submit">Save</button>
  </form>
</template>
```

Parent component:

``` vue
<script setup>
import StudentForm from '../components/StudentForm.vue'

function saveStudent(name) {
  console.log('Parent received:', name)
}
</script>

<template>
  <StudentForm @save="saveStudent" />
</template>
```

`emit('save', name.value)` emits a custom event named `save` and passes
the value. The parent listens using `@save` and receives the payload in
its handler.

### Props vs emits

  Props                      Emits
  -------------------------- ----------------------------------
  Parent to child            Child to parent
  Pass data                  Notify the parent about an event
  `defineProps()`            `defineEmits()`
  Example: student to edit   Example: `save` with form data

------------------------------------------------------------------------

## 11. `computed()` --- derived reactive values

`computed()` creates a reactive value derived from other reactive
values. Vue caches its result until a reactive dependency used by the
computation changes.

### Example: search/filter students

``` vue
<script setup>
import { ref, computed } from 'vue'

const search = ref('')

const students = ref([
  { id: '1', name: 'Arun' },
  { id: '2', name: 'Priya' },
  { id: '3', name: 'Kumar' }
])

const filteredStudents = computed(() => {
  return students.value.filter(s =>
    s.name.toLowerCase().includes(search.value.toLowerCase())
  )
})
</script>

<template>
  <input v-model="search" placeholder="Search students">

  <ul>
    <li v-for="s in filteredStudents" :key="s.id">
      {{ s.name }}
    </li>
  </ul>
</template>
```

When the user types, `search` changes and the filtered list updates
automatically.

### Understand the expression

``` javascript
const filteredStudents = computed(() => {
  return students.value.filter(s =>
    s.name.toLowerCase().includes(search.value.toLowerCase())
  )
})
```

1.  `computed(...)` creates a computed ref.
2.  `() => { ... }` is the function Vue evaluates to obtain the derived
    value.
3.  `students.value.filter(...)` creates an array containing only
    matching students.
4.  `s => ...` is a callback called for each student.
5.  `toLowerCase()` makes the search case-insensitive.
6.  `includes(...)` checks whether the student name contains the search
    text.

When `search` is an empty string, every name includes it, so all
students match.

### Why not use a normal function?

You can write:

``` javascript
function filterStudents() {
  return students.value.filter(s =>
    s.name.toLowerCase().includes(search.value.toLowerCase())
  )
}
```

And call it in the template:

``` html
<li v-for="s in filterStudents()" :key="s.id">
  {{ s.name }}
</li>
```

This can work, but the function is called whenever Vue renders that part
of the template. It does not cache the result between renders.

With `computed()`, Vue caches the derived result and recalculates it
when a reactive dependency changes. That makes `computed()` ideal for
filtered lists, totals and other derived values.

Do not confuse these with a one-time assignment:

``` javascript
const filteredStudents = students.value.filter(s =>
  s.name.toLowerCase().includes(search.value.toLowerCase())
)
```

This creates an ordinary array once. It will not automatically
recalculate when `search` changes.

### `computed()` vs normal function

  -----------------------------------------------------------------------
  Approach                Reacts to changes       Caches result
  ----------------------- ----------------------- -----------------------
  `computed()`            Yes, based on reactive  Yes
                          dependencies            

  Function called from    Called again when       No
  template                relevant rendering      
                          occurs                  

  Ordinary array assigned No automatic            Stores the original
  once                    recalculation           result
  -----------------------------------------------------------------------

Use `computed()` for values. Use normal functions for actions such as
saving, deleting or navigating.

------------------------------------------------------------------------

## 12. `watch()` --- run code when reactive data changes

`watch()` observes a reactive source and calls a callback when the
source changes.

### 12.1 Basic example

``` javascript
import { ref, watch } from 'vue'

const count = ref(0)

watch(count, (newValue, oldValue) => {
  console.log('New:', newValue)
  console.log('Old:', oldValue)
})
```

If `count.value` changes from `0` to `5`, the callback receives
`newValue = 5` and `oldValue = 0`.

-   First argument: what to watch.
-   Second argument: callback to run.
-   `newValue`: current value.
-   `oldValue`: previous value.

### 12.2 Watch a property of a reactive object

``` javascript
import { reactive, watch } from 'vue'

const form = reactive({
  name: '',
  department: ''
})

watch(
  () => form.name,
  (newName, oldName) => {
    console.log(newName, oldName)
  }
)
```

`watch()` expects a ref, reactive object, getter function or an array of
valid sources. To watch one property of a reactive object, use a getter
such as `() => form.name`.

### 12.3 Why use `watch()` in a CRUD app?

For example, when the student ID changes, you might want to fetch the
corresponding record.

``` javascript
const studentId = ref('101')
const student = ref(null)

watch(studentId, async (newId) => {
  const response = await axios.get(
    `http://localhost:8080/api/students/${newId}`
  )
  student.value = response.data
}, { immediate: true })
```

The callback sends a GET request when the ID changes. `immediate: true`
also runs it once when the watcher is first created.

In a real application, also handle loading, request failures and the
possibility of an empty ID.

### 12.4 The important props-to-form watcher

This pattern is useful when a reusable form receives a student from its
parent:

``` javascript
watch(
  () => props.student,
  (student) => {
    form.name = student?.name || ''
    form.email = student?.email || ''
    form.department = student?.department || ''
  },
  { immediate: true }
)
```

Understand each part:

**Source: `() => props.student`**

This getter tells Vue to watch the `student` prop. If the parent
replaces the student object, the watcher can run again.

**Callback: `(student) => { ... }`**

The `student` parameter is the new value passed to the callback. The
callback copies its fields into the form.

**Optional chaining: `student?.name`**

This reads `student.name` only if `student` is not `null` or
`undefined`.

**Fallback: `|| ''`**

If the name is missing or another falsy value, the form field becomes an
empty string.

**Option: `{ immediate: true }`**

The callback runs once when the watcher is created, even if the prop has
not changed. This initializes the form with the existing student data
immediately.

The form uses its own local reactive object, so editing it does not
directly mutate the prop.

Example:

``` javascript
const props = defineProps({
  student: {
    type: Object,
    default: () => ({
      name: '',
      email: '',
      department: ''
    })
  }
})

const form = reactive({
  name: '',
  email: '',
  department: ''
})
```

When the parent first supplies Arun, the watcher copies Arun's values
into the form. If the parent later supplies Priya, the watcher copies
Priya's values.

This watcher observes replacement of the student object. If you also
need to react to nested mutations on the same object, consider whether
deep watching is necessary; a deep watcher can be more expensive, and
nested mutations may give the callback the same object reference as both
old and new values.

### 12.5 `watch()` vs `computed()` vs `onMounted()`

  -----------------------------------------------------------------------
  API                     Purpose                 Example
  ----------------------- ----------------------- -----------------------
  `watch()`               Perform an effect when  Fetch details when ID
                          a source changes        changes

  `computed()`            Derive a value from     Filter students
                          reactive state          

  `onMounted()`           Run code when the       Fetch the initial list
                          component is mounted    
  -----------------------------------------------------------------------

Remember:

-   **watch:** something changed, so do something.
-   **computed:** something changed, so recalculate a value.
-   **onMounted:** the component appeared, so initialize or fetch data.

------------------------------------------------------------------------

## 13. Vue Router: normal routing and child routing

Vue Router changes the displayed page without reloading the whole
application.

Install if needed:

``` bash
npm install vue-router
```

### 13.1 Define routes: `src/router/index.js`

``` javascript
import { createRouter, createWebHistory } from 'vue-router'

import StudentLayout from '../views/StudentLayout.vue'
import StudentList from '../views/StudentList.vue'
import StudentCreate from '../views/StudentCreate.vue'
import StudentDetails from '../views/StudentDetails.vue'

const routes = [
  {
    path: '/',
    redirect: '/students'
  },
  {
    path: '/students',
    component: StudentLayout,
    children: [
      {
        path: '',
        name: 'student-list',
        component: StudentList
      },
      {
        path: 'create',
        name: 'student-create',
        component: StudentCreate
      },
      {
        path: ':id',
        name: 'student-details',
        component: StudentDetails,
        props: true
      }
    ]
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

The router is registered in `main.js` using `app.use(router)`.

### 13.2 What do these URLs mean?

  URL                  Displayed page
  -------------------- ----------------------------------------
  `/`                  Redirects to `/students`
  `/students`          Student list
  `/students/create`   Create form
  `/students/123`      Student details/edit page for ID `123`

The colon in `:id` indicates a dynamic route parameter.

### 13.3 What is child routing?

A child route is rendered inside a `<RouterView />` in its parent layout
component.

`src/views/StudentLayout.vue`:

``` vue
<template>
  <div>
    <h1>Student Management</h1>

    <nav>
      <RouterLink to="/students">All Students</RouterLink>
      |
      <RouterLink to="/students/create">Add Student</RouterLink>
    </nav>

    <hr>

    <RouterView />
  </div>
</template>
```

The route `/students` displays `StudentLayout.vue`. Its `<RouterView />`
then displays the matching child page. The heading and navigation remain
while the child page changes.

### 13.4 `RouterLink`, `RouterView`, `useRoute()` and `useRouter()`

-   `<RouterLink>` navigates to a route without a full-page reload.
-   `<RouterView>` displays the component matched by the current route.
-   `useRoute()` reads the current route, including parameters.
-   `useRouter()` provides programmatic navigation methods such as
    `push()`.

Example:

``` vue
<script setup>
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()

const id = route.params.id

function goBack() {
  router.push('/students')
}
</script>

<template>
  <h2>Student ID: {{ id }}</h2>
  <button @click="goBack">Back</button>
</template>
```

### 13.5 Pass route parameters as props

The route configuration has `props: true`:

``` javascript
{
  path: ':id',
  component: StudentDetails,
  props: true
}
```

This passes the route parameter as a prop. In the component:

``` javascript
const props = defineProps({
  id: String
})
```

You can then use `props.id`. This is an alternative to using
`useRoute()` to read `route.params.id`.

------------------------------------------------------------------------

## 14. Pinia using the updated Setup Store syntax

This guide uses the **Setup Store** syntax, not the older Options Store
structure with `state`, `getters` and `actions` object properties.

A Setup Store looks like a Vue Composition API function:

-   `ref()` defines state.
-   `computed()` defines derived values (getters).
-   Functions define actions.
-   The returned object exposes state, getters and actions to
    components.

### 14.1 What Pinia does

Pinia provides shared state that multiple components can access. For
example, `StudentList.vue` can display the students while
`StudentCreate.vue` can add one using the same store.

Without a shared store, you might end up duplicating data and API logic
in several components.

### 14.2 Complete store: `src/stores/studentStore.js`

Axios calls are directly inside the store. There is no separate API
services file.

``` javascript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import axios from 'axios'

const API_URL = 'http://localhost:8080/api/students'

export const useStudentStore = defineStore('student', () => {
  // STATE
  const students = ref([])
  const student = ref(null)
  const loading = ref(false)
  const error = ref('')

  // GETTERS (computed values)
  const totalStudents = computed(() => students.value.length)

  const cseStudents = computed(() =>
    students.value.filter(s => s.department === 'CSE')
  )

  // ACTION: READ ALL
  async function fetchStudents() {
    loading.value = true
    error.value = ''

    try {
      const response = await axios.get(API_URL)
      students.value = response.data
    } catch (e) {
      error.value = 'Failed to fetch students'
      throw e
    } finally {
      loading.value = false
    }
  }

  // ACTION: READ ONE
  async function fetchStudent(id) {
    error.value = ''

    try {
      const response = await axios.get(`${API_URL}/${id}`)
      student.value = response.data
      return response.data
    } catch (e) {
      error.value = 'Failed to fetch student'
      throw e
    }
  }

  // ACTION: CREATE
  async function addStudent(newStudent) {
    error.value = ''

    try {
      const response = await axios.post(API_URL, newStudent)
      students.value.push(response.data)
      return response.data
    } catch (e) {
      error.value = 'Failed to add student'
      throw e
    }
  }

  // ACTION: UPDATE
  async function updateStudent(id, updatedStudent) {
    error.value = ''

    try {
      const response = await axios.put(
        `${API_URL}/${id}`,
        updatedStudent
      )

      // This assumes the PUT response contains the updated student.
      const savedStudent = response.data
      const index = students.value.findIndex(s => s.id === id)

      if (index !== -1) {
        students.value[index] = savedStudent
      }

      student.value = savedStudent
      return savedStudent
    } catch (e) {
      error.value = 'Failed to update student'
      throw e
    }
  }

  // ACTION: DELETE
  async function deleteStudent(id) {
    error.value = ''

    try {
      await axios.delete(`${API_URL}/${id}`)
      students.value = students.value.filter(s => s.id !== id)

      if (student.value?.id === id) {
        student.value = null
      }
    } catch (e) {
      error.value = 'Failed to delete student'
      throw e
    }
  }

  return {
    // state
    students,
    student,
    loading,
    error,

    // getters
    totalStudents,
    cseStudents,

    // actions
    fetchStudents,
    fetchStudent,
    addStudent,
    updateStudent,
    deleteStudent
  }
})
```

### 14.3 Explain each part

**Store definition**

``` javascript
export const useStudentStore = defineStore('student', () => {
  // ...
})
```

-   `defineStore()` creates a Pinia store.
-   `'student'` is the unique store ID.
-   The arrow function defines the Setup Store.
-   `useStudentStore()` is the function components call to access the
    store.

**State**

``` javascript
const students = ref([])
const student = ref(null)
const loading = ref(false)
const error = ref('')
```

-   `students`: list returned by the backend.
-   `student`: currently selected student.
-   `loading`: whether a list fetch is in progress.
-   `error`: error message for the UI.

In the Setup Store's JavaScript functions, refs are accessed using
`.value`.

**Getters**

``` javascript
const totalStudents = computed(() => students.value.length)
```

This automatically calculates the number of students. In a component,
access it as `store.totalStudents`, not `store.totalStudents()`.

**Actions**

Actions are the functions that perform work, such as calling Axios and
changing state.

-   `fetchStudents()` retrieves all records.
-   `fetchStudent(id)` retrieves one record.
-   `addStudent(data)` creates a record.
-   `updateStudent(id, data)` updates a record.
-   `deleteStudent(id)` deletes a record.

**Return object**

The return object exposes the state, getters and actions to components.
If a value or function is not returned, it is not directly available
through the store instance.

### 14.4 Important backend response assumptions

The example assumes:

1.  GET all returns an array in `response.data`.
2.  GET one returns one student object.
3.  POST returns the created student object.
4.  PUT returns the updated student object.
5.  DELETE succeeds when the request completes.
6.  Student identifiers are returned as `id`.

If your backend returns `_id`, change the ID references accordingly. If
PUT returns no body (for example, HTTP 204), do not use `response.data`
as the saved record. Update local state using the submitted object or
refetch the list instead.

The frontend endpoint must match the backend controller mapping exactly.

------------------------------------------------------------------------

## 15. Use Pinia in the student list page

`src/views/StudentList.vue`:

``` vue
<script setup>
import { onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useStudentStore } from '../stores/studentStore'

const store = useStudentStore()
const router = useRouter()

onMounted(() => {
  store.fetchStudents()
})

function editStudent(student) {
  router.push(`/students/${student.id}`)
}

async function removeStudent(id) {
  if (!confirm('Are you sure you want to delete this student?')) {
    return
  }

  try {
    await store.deleteStudent(id)
  } catch (e) {
    alert('Unable to delete student')
  }
}
</script>

<template>
  <section>
    <h2>Student List</h2>

    <RouterLink to="/students/create">Add Student</RouterLink>

    <p v-if="store.loading">Loading students...</p>
    <p v-else-if="store.error">{{ store.error }}</p>

    <table v-else>
      <thead>
        <tr>
          <th>Name</th>
          <th>Email</th>
          <th>Department</th>
          <th>Actions</th>
        </tr>
      </thead>

      <tbody>
        <tr v-for="s in store.students" :key="s.id">
          <td>{{ s.name }}</td>
          <td>{{ s.email }}</td>
          <td>{{ s.department }}</td>
          <td>
            <button @click="editStudent(s)">Edit</button>
            <button @click="removeStudent(s.id)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <p v-if="!store.loading && !store.error && store.students.length === 0">
      No students found.
    </p>

    <p>Total students: {{ store.totalStudents }}</p>
  </section>
</template>
```

### What happens?

-   `useStudentStore()` obtains the shared store.
-   `onMounted()` calls `fetchStudents()` when the page mounts.
-   `store.students` contains the list from the API.
-   `v-for` renders one row per student.
-   The Edit button navigates to the student's dynamic route.
-   The Delete button confirms and calls the store action.
-   Pinia updates the list after successful deletion, and Vue updates
    the table.

`onMounted()` is appropriate for initial loading. If a component can
remain mounted while its route parameter changes, watch the route
parameter or use a route-aware loading pattern as well.

------------------------------------------------------------------------

## 16. Create student page

`src/views/StudentCreate.vue`:

``` vue
<script setup>
import { reactive, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useStudentStore } from '../stores/studentStore'

const store = useStudentStore()
const router = useRouter()

const form = reactive({
  name: '',
  email: '',
  department: ''
})

const saving = ref(false)
const error = ref('')

async function saveStudent() {
  saving.value = true
  error.value = ''

  try {
    await store.addStudent({ ...form })
    router.push('/students')
  } catch (e) {
    error.value = store.error || 'Unable to save student'
  } finally {
    saving.value = false
  }
}
</script>

<template>
  <section>
    <h2>Add Student</h2>

    <form @submit.prevent="saveStudent">
      <label>Name</label>
      <input v-model.trim="form.name" required>

      <label>Email</label>
      <input v-model.trim="form.email" type="email" required>

      <label>Department</label>
      <select v-model="form.department" required>
        <option value="" disabled>Select department</option>
        <option value="CSE">CSE</option>
        <option value="ECE">ECE</option>
        <option value="IT">IT</option>
      </select>

      <button type="submit" :disabled="saving">
        {{ saving ? 'Saving...' : 'Save Student' }}
      </button>
    </form>

    <p v-if="error">{{ error }}</p>
  </section>
</template>
```

Explanation:

1.  `reactive()` creates the form object.
2.  `v-model.trim` binds inputs and trims surrounding whitespace.
3.  `@submit.prevent` prevents a page reload and invokes
    `saveStudent()`.
4.  `store.addStudent({ ...form })` sends a copy of the form object to
    Pinia.
5.  Pinia calls Axios POST and updates its state.
6.  `router.push()` returns to the list after success.
7.  `try/catch/finally` displays failures and resets the saving flag.

The spread syntax `{ ...form }` creates a shallow copy of the form's
properties in a normal object.

------------------------------------------------------------------------

## 17. Update student page

This page uses the ID from `/students/:id`, fetches the existing student
and pre-fills the form.

`src/views/StudentDetails.vue`:

``` vue
<script setup>
import { onMounted, reactive, ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useStudentStore } from '../stores/studentStore'

const route = useRoute()
const router = useRouter()
const store = useStudentStore()

const form = reactive({
  name: '',
  email: '',
  department: ''
})

const loading = ref(true)
const saving = ref(false)
const error = ref('')

onMounted(async () => {
  try {
    const student = await store.fetchStudent(route.params.id)

    form.name = student.name || ''
    form.email = student.email || ''
    form.department = student.department || ''
  } catch (e) {
    error.value = store.error || 'Unable to load student'
  } finally {
    loading.value = false
  }
})

async function saveChanges() {
  saving.value = true
  error.value = ''

  try {
    await store.updateStudent(route.params.id, { ...form })
    router.push('/students')
  } catch (e) {
    error.value = store.error || 'Unable to update student'
  } finally {
    saving.value = false
  }
}
</script>

<template>
  <section>
    <h2>Edit Student</h2>

    <p v-if="loading">Loading student...</p>

    <form v-else @submit.prevent="saveChanges">
      <label>Name</label>
      <input v-model.trim="form.name" required>

      <label>Email</label>
      <input v-model.trim="form.email" type="email" required>

      <label>Department</label>
      <select v-model="form.department" required>
        <option value="CSE">CSE</option>
        <option value="ECE">ECE</option>
        <option value="IT">IT</option>
      </select>

      <button type="submit" :disabled="saving">
        {{ saving ? 'Updating...' : 'Update Student' }}
      </button>
    </form>

    <p v-if="error">{{ error }}</p>
  </section>
</template>
```

### Update flow

1.  The user opens `/students/123`.
2.  `useRoute()` exposes `route.params.id`.
3.  `fetchStudent(id)` sends GET `/api/students/123`.
4.  The returned record fills the form.
5.  The user edits the fields.
6.  `updateStudent(id, form)` sends PUT.
7.  Pinia updates the local list if the backend returns the updated
    record.
8.  The page navigates back to the list.

If your API returns `_id` instead of `id`, use `_id` consistently. If a
route parameter may change while the component stays mounted, watch the
parameter and refetch the record.

------------------------------------------------------------------------

## 18. Optional: reusable `StudentForm.vue` with props and emits

You can separate the form UI from the page logic. This is useful if the
same form is used for both creating and editing.

`src/components/StudentForm.vue`:

``` vue
<script setup>
import { reactive, watch } from 'vue'

const props = defineProps({
  student: {
    type: Object,
    default: () => ({
      name: '',
      email: '',
      department: ''
    })
  },
  buttonText: {
    type: String,
    default: 'Save'
  }
})

const emit = defineEmits(['save'])

const form = reactive({
  name: '',
  email: '',
  department: ''
})

watch(
  () => props.student,
  (student) => {
    form.name = student?.name || ''
    form.email = student?.email || ''
    form.department = student?.department || ''
  },
  { immediate: true }
)

function submitForm() {
  emit('save', { ...form })
}
</script>

<template>
  <form @submit.prevent="submitForm">
    <label>Name</label>
    <input v-model.trim="form.name" required>

    <label>Email</label>
    <input v-model.trim="form.email" type="email" required>

    <label>Department</label>
    <select v-model="form.department" required>
      <option value="">Select department</option>
      <option value="CSE">CSE</option>
      <option value="ECE">ECE</option>
      <option value="IT">IT</option>
    </select>

    <button type="submit">{{ buttonText }}</button>
  </form>
</template>
```

The parent can use it like this:

``` vue
<StudentForm @save="saveStudent" button-text="Create" />

<StudentForm
  :student="existingStudent"
  @save="updateStudent"
  button-text="Update"
/>
```

-   `student` is passed as a prop.
-   `buttonText` customizes the button label.
-   `@save` listens for the child event.
-   `emit('save', { ...form })` sends form data to the parent.
-   `watch()` copies the supplied student into the local form.
-   `immediate: true` initializes the form when the watcher is created.

This is an alternative design. If you use the reusable component, keep
API operations in the parent or Pinia store, rather than duplicating
them in the form component.

------------------------------------------------------------------------

## 19. Styling the CRUD pages

Create `src/assets/main.css`:

``` css
body {
  font-family: Arial, sans-serif;
  margin: 30px;
  background: #f5f5f5;
}

h1, h2 {
  color: #222;
}

input, select {
  display: block;
  width: 100%;
  max-width: 400px;
  padding: 10px;
  margin: 8px 0 16px;
  box-sizing: border-box;
}

button {
  padding: 8px 14px;
  margin: 4px;
  cursor: pointer;
}

table {
  width: 100%;
  border-collapse: collapse;
  background: white;
}

th, td {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: left;
}

th {
  background: #e8e8e8;
}
```

Import the file in `main.js`:

``` javascript
import './assets/main.css'
```

This provides basic styling. You can later use Flexbox or CSS Grid for
responsive layouts.

------------------------------------------------------------------------

## 20. Common problems and debugging

  -----------------------------------------------------------------------
  Error/problem                       What to check
  ----------------------------------- -----------------------------------
  Axios `Network Error`               Is Spring Boot running? Is the port
                                      correct?

  CORS error                          Does Spring Boot allow the frontend
                                      origin?

  `404 Not Found`                     Does the URL match the controller's
                                      request mapping?

  `400 Bad Request`                   Do JSON property names and data
                                      types match the Java model?

  `405 Method Not Allowed`            Are you using GET, POST, PUT or
                                      DELETE correctly?

  List remains empty                  Does GET return an array? Are you
                                      assigning `response.data`?

  Create succeeds but no row appears  Does POST return the created
                                      object? Otherwise refetch the list.

  Update fails                        Does PUT expect an ID in the path
                                      or body? Does its response contain
                                      an object?

  Delete fails                        Does the controller accept DELETE
                                      for that endpoint?

  Student ID is undefined             Check whether the backend returns
                                      `id` or `_id`.

  Inputs don't update                 Check `v-model` and reactive state.

  Watcher doesn't run                 Confirm the source is reactive and
                                      that the watched value actually
                                      changes.

  Form starts empty in edit mode      Check the GET-by-ID request and
                                      whether the watcher uses
                                      `immediate: true`.

  `this` is undefined in a function   In a Setup Store, use returned
                                      state variables; arrow functions do
                                      not have their own `this`.
  -----------------------------------------------------------------------

### CORS example

If the frontend runs at `http://localhost:5173`, Spring Boot may need to
allow that origin.

``` java
@CrossOrigin(origins = "http://localhost:5173")
@RestController
@RequestMapping("/api/students")
public class StudentController {
    // CRUD endpoints
}
```

This is a simple development example. If Spring Security is enabled,
configure CORS there as appropriate too.

------------------------------------------------------------------------

## 21. Quick revision sheet

  -----------------------------------------------------------------------
  Concept                             Remember
  ----------------------------------- -----------------------------------
  `ref()`                             Reactive value; use `.value` in
                                      JavaScript.

  `reactive()`                        Reactive object; access properties
                                      directly.

  `v-model`                           Connects inputs to state.

  `v-for`                             Repeats elements for arrays.

  `v-if`                              Conditional rendering.

  `@click`                            Handles click events.

  `defineProps()`                     Parent passes data to child.

  `defineEmits()`                     Child emits events to parent.

  `computed()`                        Cached derived reactive value.

  `watch()`                           Runs a callback when a reactive
                                      source changes.

  `onMounted()`                       Runs after component mounting.

  `useRoute()`                        Reads current route and parameters.

  `useRouter()`                       Navigates programmatically.

  `<RouterLink>`                      Navigation link.

  `<RouterView>`                      Renders matched route component.

  Pinia Setup Store                   State uses refs, getters use
                                      computed, actions are functions.

  Axios                               Sends HTTP requests to Spring Boot.

  GET                                 Read.

  POST                                Create.

  PUT                                 Update.

  DELETE                              Delete.
  -----------------------------------------------------------------------

### The architecture to remember

``` text
View/component
  ├── Form inputs and table
  ├── Calls Pinia actions
  └── Uses router for navigation
          ↓
Pinia Setup Store
  ├── State: ref()
  ├── Getters: computed()
  ├── Actions: functions
  └── Axios calls directly here
          ↓ HTTP + JSON
Spring Boot REST API
          ↓
MongoDB
```