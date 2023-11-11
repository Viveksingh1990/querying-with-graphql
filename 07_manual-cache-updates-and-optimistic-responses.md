Manual cache updates and optimistic responses
=============================================

In the previous lesson we learned how to change the data on the GraphQL server with mutations, and how they are able to automatically update the Apollo cache. Unfortunately, automatic update is not always the case. This works when we have an existing entity (in our case, it’s a `Book` and we update some properties of it), but what happens if we want to create a new entity and add it to the list? How do we handle Apollo cache changes and UI updates? We will answer all these questions in this lesson.

* * *

Writing a mutation to add a new book
------------------------------------

Before we start adding any new code to our Vue application, let’s first shape a mutation to add a new book in our GraphQL playground.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1644891631381.jpg?alt=media&token=d877b7d6-ccce-4c70-acb9-9cbbceb79f99](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1644891631381.jpg?alt=media&token=d877b7d6-ccce-4c70-acb9-9cbbceb79f99)

As we can see on the screenshot above, the `addBook` mutation expects a `BookInput` type: we have to provide book title, author, and a year as a bare minimum (you can see these fields are required because they have an exclamation mark). We don’t need to provide any ID — the unique identifier will be generated for us on the server.

This mutation will return us a newly created book. We can see this by return type of `Book!`

Let’s write a mutation in the playground and see what is returned in the response:

    mutation AddBook {
      addBook(input: {
        title: "Test Book",
        author: "Natalia",
        year: 2021})
      {
        id
        title
        author
        year
        rating
      }
    }
    

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1644891631382.jpg?alt=media&token=bb70d32c-a41c-4fcf-8bd2-5103d3a156ad](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1644891631382.jpg?alt=media&token=bb70d32c-a41c-4fcf-8bd2-5103d3a156ad)

Now, if we run `allBooks` query again, we will see a new book in the list:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.opt.1644891637769.jpg?alt=media&token=736644f1-02de-44a3-9448-549a20af87f2](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.opt.1644891637769.jpg?alt=media&token=736644f1-02de-44a3-9448-549a20af87f2)

Now that we have nicely shaped a mutation, we can move on to writing a component to call this mutation.

* * *

Adding a new book form component
--------------------------------

Back in our project, let’s create a new file `AddBook.vue` in the components folder. We want this new component to be a form containing five fields: `title`, `description`, `year`, `author` and `rating`. We won’t care about form validation much in this lesson but we will make `title` , `year` and `author` required fields to match the required variables in the `addBook` GraphQL mutation. Also, we want to have two buttons: one will submit and close the form, and the second one is a “Cancel” button just to close the form.

Our component’s template will look like this:

📃 **components/AddBook.vue**

    <template>
      <form>
        <label for="title">
          Title
          <input type="text" id="title" required />
        </label>
        <label for="author">
          Author
          <input type="text" id="author" required />
        </label>
        <label for="description">
          Description (optional)
          <input type="text" id="description" />
        </label>
        <label for="year">
          Year
          <input type="number" id="year" required />
        </label>
        <label for="rating">
          Rating (optional)
          <input type="number" id="rating" />
        </label>
        <button type="submit">Submit</button>
        <button type="reset" @click="$emit('closeForm')">Close form</button>
      </form>
    </template>
    

As you can see, we already have an event to emit when we want to cancel the book form.

Now, in the `<script>` part of the component, let’s add a `newBook` reactive object and a placeholder method `addBook`. For now, we want to just log the `newBook` in the browser console and emit the event to close the form:

📃 **components/AddBook.vue**

    <script>
    import { reactive } from 'vue'
    export default {
      emits: ['closeForm'],
      setup(_, { emit }) {
        const newBook = reactive({
          title: '',
          author: '',
          year: '',
          rating: '',
          description: '',
        })
    
        const addBook = () => {
          console.log(newBook)
          emit('closeForm')
        }
    
        return {
          addBook,
          newBook,
        }
      },
    }
    </script>
    

Let’s walk through this component checking what’s going on here.

    import { reactive } from 'vue'
    

In this component, we want to group reactive properties in one object and (unlike in the `EditRating` component) we will do it with a `reactive` method instead of `ref`. `reactive` can be handy when you only want to update fields of the object but never replace the object itself.

      emits: ['closeForm'],
      setup(_, { emit }) {/* setup code */}
    

We declare that the component will emit an event to close the form. In `setup`, the first argument is `props` and we don’t need any passed prop here, so we replaced it with underscore. It’s a common practice for the cases we don’t need for example a first argument of a method but in order to access the second argument, we still need to pass both. In such cases, underscore says “this is something we are not going to use here”.

The second argument is a `context` object and we only need its `emit` method, so we use ES6 destructuring to retrieve it.

Now, let’s add `v-model` to every input field and a `submit` event for the whole form. We want our `v-model` to use the `.trim` modifier so that any whitespaces on the beginning/end of the value will be trimmed. For the `@submit` listener, we’ll use a `prevent` modifier to prevent a native form submit and call the `addBook` method instead:

📃 **components/AddBook.vue**

    <template>
      <!-- Submit event -->
      <form @submit.prevent="addBook"> 
        <label for="title">
          Title
          <input type="text" id="title" required v-model.trim="newBook.title" />
        </label>
        <label for="author">
          Author
          <input type="text" id="author" required v-model.trim="newBook.author" />
        </label>
        <label for="description">
          Description (optional)
          <input type="text" id="description" v-model.trim="newBook.description" />
        </label>
        <label for="year">
          Year
          <input type="number" id="year" required v-model.number="newBook.year" />
        </label>
        <label for="rating">
          Rating (optional)
          <input type="number" id="rating" v-model.number="newBook.rating" />
        </label>
        <button type="submit">Submit</button>
        <button type="reset" @click="$emit('closeForm')">Close form</button>
      </form>
    </template>
    

Now, let’s modify our `App.vue` component to include `AddBook`. First, let’s import the `AddBook` component and add it to the `components` property:

📃 **App.vue**

    import AddBook from './components/AddBook'
    
    export default {
      name: 'App',
      components: {
        EditRating,
        AddBook,
      },
      /* ... */
    }
    

Also, we will need a new reactive property to define when we show the new book form and when we hide it:

📃 **App.vue**

    setup() {
      /* ... */
      const showNewBookForm = ref(false)
      /* ... */
    
      return { books, searchTerm, loading, error, activeBook, showNewBookForm }
    },
    

Finally, we can add the new component to the template:

📃 **App.vue**

    <template>
      <div>
        <div>
          <button v-if="!showNewBookForm" @click="showNewBookForm = true">
            Add a new book
          </button>
          <AddBook v-if="showNewBookForm" @closeForm="showNewBookForm = false" />
        </div>
        <input type="text" v-model="searchTerm" />
        <p v-if="loading">Loading...</p>
        <p v-else-if="error">Something went wrong! Please try again</p>
        <template v-else>
          <!-- ... -->
        </template>
      </div>
    </template>
    

Now that we’ve got these steps to work, we are ready to replace the logic inside the `addBook` method so that it actually calls a GraphQL mutation instead of just logging the new book to the console.

* * *

Calling a mutation from the Vue component
-----------------------------------------

Let’s start with creating a new file in the `graphql` folder to store our `addBook` mutation. We will name it `addBook.mutation.gql`and copy-paste the mutation from the GraphQL playground to this file:

📃 **addBook.mutation.graphql**

    mutation AddBook {
      addBook(input: {
        title: "Test Book",
        author: "Natalia",
        year: 2021})
      {
        id
        title
        author
        year
        rating
      }
    }
    

Let’s make a few tweaks here. First of all, we want to pass a new book as a variable here. When checking the GraphQL docs on the playground, we can see that the mutation expects a variable `input`:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1644891645352.jpg?alt=media&token=c45416b5-7182-436c-888b-731f9ee33817](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1644891645352.jpg?alt=media&token=c45416b5-7182-436c-888b-731f9ee33817)

Let’s add this variable to the mutation:

📃 **addBook.mutation.graphql**

    mutation AddBook($input: BookInput!) {
      addBook(input: $input)
      {
        id
        title
        author
        year
        rating
      }
    }
    

Also, we can reuse our `BookFragment` we created earlier in the course to render book properties here:

📃 **addBook.mutation.graphql**

    #import './book.fragment.gql'
    
    mutation AddBook($input: BookInput!) {
      addBook(input: $input) {
        ...BookFragment
      }
    }
    

Now we can import this mutation to our `AddBook.vue` component, as well as the `useMutation` method from VueApollo.

📃 **AddBook.vue**

    import ADD_BOOK_MUTATION from '../graphql/addBook.mutation.gql'
    import { useMutation } from '@vue/apollo-composable'
    

We will follow the same steps as for the `updateBook` mutation in the previous lesson: first, we expose the `mutate` method and `onDone` hook from `useMutation`.

📃 **AddBook.vue**

    setup(_, { emit }) {
      const newBook = reactive({
        title: '',
        author: '',
        year: '',
        rating: '',
        description: '',
      })
      
      const { mutate, loading, error, onDone } = useMutation()
      
      const addBook = () => {
        console.log(newBook)
        emit('closeForm')
      }
      
      return {
        addBook,
        newBook,
      }
    },
    

We need to pass our imported mutation as a first parameter to `useMutation`, and as a second parameter we will pass a callback, returning the `variables` object. We need to pass an `input` variable, and in our case it will be equal to the `newBook` object:

📃 **AddBook.vue**

    const { mutate, loading, error, onDone } = useMutation(
      ADD_BOOK_MUTATION,
      () => ({
        variables: {
          input: newBook,
        },
      })
    )
    

Now, we can alias `mutate` as `addBook`, remove the old placeholder `addBook` method, and emit the `closeForm` event in `onDone`:

    setup(_, { emit }) {
      const newBook = reactive({
        title: '',
        author: '',
        year: '',
        rating: '',
        description: '',
      })
    
      const { mutate: addBook, onDone } = useMutation(
        ADD_BOOK_MUTATION,
        () => ({
          variables: {
            input: newBook,
          },
        })
      )
    
      onDone(() => emit('closeForm'))
    
      return {
        addBook,
        newBook,
      }
    },
    

Let’s check the network tab to see if our GraphQL call is successful!

Yes! We can see the mutation is being sent to the server and returns a successful response! Let’s make a small modification and remove the requirement for at least to characters in the `searchTerm` to show the full list of books:

📃 **App.vue**

    const { result, loading, error } = useQuery(
          ALL_BOOKS_QUERY,
          () => ({
            search: searchTerm.value,
          }),
          () => ({
            debounce: 500,
            // enabled: searchTerm.value.length > 2,
          })
        )
    

After commenting this line and refreshing the page, we should see two `Test Book` entries in the list:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F7.1644891651550.jpg?alt=media&token=b15b5b0f-30d8-4729-9bd7-f642f91c7cc1](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F7.1644891651550.jpg?alt=media&token=b15b5b0f-30d8-4729-9bd7-f642f91c7cc1)

However, if we try to add a new book, we will see that the list is not automatically updating like it was with updating the rating:

This happens because we are creating a new entity, and for our Apollo cache, it’s a new ID so there is nothing to update. We need to work with the cache manually to update the data there, and this will trigger the page re-render.

* * *

Updating Apollo cache manually
------------------------------

In Apollo cache, stored data is associated with queries. When data specific to the certain query is updated, this query _returns the new result_. It’s similar in a way to Vuex getters — queries serve to return a certain part of the cache, and with updating this cache the result is recalculated.

To update this data, we will need to _read_ the query first from the cache with Apollo’s `readQuery` method. Then we will _modify_ the data, and we need to remember to always do this in an immutable way, with creating a new entity rather than mutating the cache. After the modification, we will write the new data back with `writeQuery` method.

To access the cache and use these two methods, we will use the mutation‘s `update` hook. First, lets add the `update` to our `addBook` mutation right after `variables`:

**components/AddBook.vue**

    const { mutate: addBook, onDone } = useMutation(
      ADD_BOOK_MUTATION,
      () => ({
        variables: {
          input: newBook,
        },
        update() {})
    )
    

In the `update` callback, the first argument is a reference to our Apollo cache. We will access it and modify the data. The second argument is the response our mutation returns from the server. Let’s inspect the browser console `Network` tab to see how this response looks:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F9.1644891657467.jpg?alt=media&token=6319dc36-48d9-4295-a483-ac41c1a30e07](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F9.1644891657467.jpg?alt=media&token=6319dc36-48d9-4295-a483-ac41c1a30e07)

We will need `addBook` here to add it to our Apollo cache `allBooks` list manually. Let’s add our arguments to the `update`:

📃 **components/AddBook.vue**

    const { mutate: addBook, onDone } = useMutation(
      ADD_BOOK_MUTATION,
      () => ({
        variables: {
          input: newBook,
        },
        update(cache, response) {})
    )
    

Let’s start with reading the `allBooks` query from the cache. In order to do so, we need to import this query into the `AddBook.vue` component:

📃 **components/AddBook.vue**

    import ALL_BOOKS_QUERY from '../graphql/allBooks.query.gql'
    

And now we can pass this as a parameter to the `readQuery` cache method:

📃 **components/AddBook.vue**

    const { mutate: addBook, onDone } = useMutation(
      ADD_BOOK_MUTATION,
      () => ({
        variables: {
          input: newBook,
        },
        update(cache, response) {
          const sourceData = cache.readQuery({ query: ALL_BOOKS_QUERY })
          console.log(sourceData)
        })
    )
    

Surprisingly, we will see `null` in the console! What did we miss?

Usually, this happens when the data shape in the cache is not matching the query we are reading (for example, the query has different fields or variables). We are sure it’s the same query, but let’s check the Apollo devtools to see variables:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F10.opt.1644891662331.jpg?alt=media&token=86d0f422-f9f0-4db8-bd4b-6e0402a1a33e](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F10.opt.1644891662331.jpg?alt=media&token=86d0f422-f9f0-4db8-bd4b-6e0402a1a33e)

There is a `search` variable we need to add to `readQuery`! Let’s pass the `searchTerm` as a prop from `App.vue` to `AddBook.vue`:

📃 **App.vue**

    <AddBook
      v-if="showNewBookForm"
      :search="searchTerm"
      @closeForm="showNewBookForm = false"
    />
    

📃 **components/AddBook.vue**

    export default {
      emits: ['closeForm'],
      props: {
        search: {
          type: String,
          required: true,
        },
      },
      setup(props, { emit }) {...},
    }
    

Now, let’s add the `search` variable to the `readQuery` arguments:

📃 **components/AddBook.vue**

    update(cache, response) {
      const sourceData = cache.readQuery({
        query: ALL_BOOKS_QUERY,
        variables: { search: props.search },
      })
      console.log(sourceData)
    },
    

After this, we can see the `allBooks` query as it’s stored in the Apollo cache. Let’s create a new `data` variable and combine the `allBooks` array with the newly added book from the response:

📃 **components/AddBook.vue**

    update(cache, response) {
      const sourceData = cache.readQuery({
        query: ALL_BOOKS_QUERY,
        variables: { search: props.search },
      })
      const data = {
        allBooks: [...sourceData.allBooks, response.data.addBook],
      }
    },
    

**Important note:** we are not pushing the new book directly to `sourceData.allBooks` to respect the cache immutability. If we try to do so, Apollo will throw a warning.

The last step is to write the new data back to the cache. Again, we will need to pass the query we want to modify, correct variables, and a new data as a `data` property:

📃 **components/AddBook.vue**

    update(cache, response) {
      const sourceData = cache.readQuery({
        query: ALL_BOOKS_QUERY,
        variables: { search: props.search },
      })
      const data = {
        allBooks: [...sourceData.allBooks, response.data.addBook],
      }
      cache.writeQuery({
        data,
        query: allBooksQuery,
        variables: { search: props.search },
      })
    },
    

Now, whenever we add a new book, we can see the list updated immediately!

* * *

Optimistic updates
------------------

Sometimes we want to show the result of the mutation on the page even before this mutation is resolved. For example, when we reorder items in the list with drag and drop, it might not be the best user experience to show the loader every single time after the reordering.

But how do we tell Apollo to show the mutation result _before_ we retrieved this result from the server? In our case of adding a new book to the list, this task is rather trivial. We _know_ the book data — we already entered this data in the form. We only lack the unique ID of the book (because it’s generated on our server) but we can show the rest of the data.

To add an optimistic response to our mutation, we can use the `optimisticResponse` property in the mutation options, right after the `update`:

📃 **components/AddBook.vue**

    const { mutate: addBook, onDone } = useMutation(ADD_BOOK_MUTATION, () => ({
      variables: {
        input: newBook,
      },
      update(cache, response) {...},
      optimisticResponse: {},
    }))
    

Now, we need to understand what should be inside the optimistic response. In order for it to work properly, this should be an object that matches an actual API response, including a `__typename` property for every nested object. What is `__typename`? This is a GraphQL type we can find in our schema.

So, for our mutation, the correct `optimisticResponse` will look like this:

📃 **components/AddBook.vue**

    optimisticResponse: {
      addBook: {
        __typename: 'Book',
        id: -1,
        ...newBook,
      },
    },
    

Here, we have the same root property as the mutation response doe: `addBook`. The response type for the mutation is `Book`, so we put this in the `__typename`. We don’t know what will be the id, so we put a `-1` as a placeholder here, and we spread the `newBook` object to populate the book properties.

How does it look in the UI?

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F12.gif?alt=media&token=f971f697-90d5-4ed2-8fd0-294870964b01](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F12.gif?alt=media&token=f971f697-90d5-4ed2-8fd0-294870964b01)

As you can see, the mutation is still not resolved (form is shown) but we already have a new book added to the list.

* * *

Let’s Revue
-----------

In this lesson , we learned how to update Apollo cache manually when we are adding a new book to the list. This is very useful for the cases when automatic updates do not apply, and gives you a bit more fine-grained control over the Apollo cache.

### Lesson Resources

##### Source Code:

*   [Server Repo](https://github.com/Code-Pop/graphql-server)
    
*   [Client Repo Starting Code](https://github.com/Code-Pop/graphql-client/tree/L8-start)
    
*   [Client Repo Ending Code](https://github.com/Code-Pop/graphql-client/tree/L8-end)
