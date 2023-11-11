Query variables, handling loading and errors
============================================

So far in this course, we have been fetching the full list of books. This was fine because our list was really short, but what if we had a thousand books and we needed to search for a certain one? For more complex queries like that, we can take advantage of GraphQL’s `search` parameter.

* * *

Filtering the book list with a query variable
---------------------------------------------

Before we integrate this feature into our actual app, let’s head into the GraphQL playground to practice. We’ll utilize the `search` parameter on the `allBooks` query.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1642010013367.jpg?alt=media&token=bd2a1e3f-8e35-4030-916b-0fd65bdfd538](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1642010013367.jpg?alt=media&token=bd2a1e3f-8e35-4030-916b-0fd65bdfd538)

In this example, we are using a _hardcoded_ value on our query. If we copied this query to our application, we would not be able to pass the search term to the query; it would be a part of the query string. To pass a dynamic value to a GraphQL query, we can use GraphQL variables.

First, let’s declare the variables our query can accept, which we can do by adding an argument to the query.

    query AllBooks($search: String) { # add argument
      allBooks(search: "train") {
        id
        title
        rating
      }
    }
    

Variables must be prefixed with `$` and we need to define a type for them. In our case, we are searching for book _titles_ so our search variable is definitely a `String`.

We can also check what is accepted as a `search` parameter:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1642010013368.jpg?alt=media&token=287c6a57-041e-4179-bb85-caa444a2b42f](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1642010013368.jpg?alt=media&token=287c6a57-041e-4179-bb85-caa444a2b42f)

Now, we can pass the `$search` variable to the `search` argument:

    query AllBooks($search: String) {
      allBooks(search: $search) { # now search is dynamic, accepting an external value
        id
        title
        rating
      }
    }
    

Finally, we can test that the passed variable is correctly applied. To do this in the playground, we can use the `Query variables` tab at the bottom, and add a string such as `"the"`:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.opt.1642010020430.jpg?alt=media&token=84604bc0-64a0-4af0-88b8-d5088a3cd9e5](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.opt.1642010020430.jpg?alt=media&token=84604bc0-64a0-4af0-88b8-d5088a3cd9e5)

As you can see, now our results are only books whose titles contain the string `"the"`.

Now that we’ve gotten this practice in the playground, let’s make use of the search in our Vue application.

* * *

Adding search to the application
--------------------------------

To add the search ability, we’ll first need to modify the `allBooks` query in our Vue application to reflect those changes we made in the playground.

Let’s add the `$search` variable here:

📃 **allBooks.query.gql**

    query AllBooks($search: String) {
      allBooks(search: $search) {
        id
        title
        rating
      }
    }
    

Now, in the `App.vue`, let’s pass a variable to the `useQuery` composable. To make a quick check if it works, first we are going to pass a hardcoded string: `"the"`.

📃 **App.vue**

     setup() {
        const { result } = useQuery(ALL_BOOKS_QUERY, { search: 'the' })
    
        return { result }
      },
    

Now when we run our app, we’ll see that the rendered result has changed:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.1642010023504.jpg?alt=media&token=82b892af-95cf-43a9-a4e5-b71c3cfd7075](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.1642010023504.jpg?alt=media&token=82b892af-95cf-43a9-a4e5-b71c3cfd7075)

* * *

Making it interactive
---------------------

So far so good. But we want to make this interactive so that we can make real-time searches for anything a user types into a search input. So let’s add a new input field and bind that field to a new reactive property.

In the template, we’ll add the `<input>` with that `v-model`:

📃 **App.vue**

    <template>
      <div>
        <input type="text" v-model="searchTerm" />
        <p v-for="book in result.allBooks" :key="book.id">
          {{ book.title }}
        </p>
      </div>
    </template>
    

And in our script section, we’ll add a reactive property called `searchTerm`:

📃 **App.vue**

    import { ref } from 'vue'
    
    export default {
      name: 'App',
      setup() {
        const searchTerm = ref('') // create a reactive property with 'ref'
        const { result } = useQuery(ALL_BOOKS_QUERY, { search: 'the' })
    
        return { result, searchTerm } // don't forget to return 'searchTerm'!
      },
    }
    

Now instead of that hardcoded string variable, let’s replace it with a `searchTerm.value` (don’t forget that we need to access `.value` when working with refs inside the `setup()`)

        const searchTerm = ref('') // create a reactive property with 'ref'
        const { result } = useQuery(ALL_BOOKS_QUERY, { search: searchTerm.value })
    

We’re almost there, but right now this is not going to work. To make the query change based upon the input variable, we need our search argument to be a _function that returns an object_ as the variable:

    const { result } = useQuery(ALL_BOOKS_QUERY, () => ({ search: searchTerm.value }))
    

Now, when we type, the new query is sent to the GraphQL API, returning the list filtered according to the `searchTerm`:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1642010108719.gif?alt=media&token=ed304871-799f-4120-b360-efe193e21a08](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1642010108719.gif?alt=media&token=ed304871-799f-4120-b360-efe193e21a08)

* * *

Handling loading state
----------------------

In real-life applications, user interfaces usually display some kind of loading state when an API request is still in progress. It’s common to create a loading flag, setting it to `true` when the query starts, and to `false` when it’s resolved.

In Apollo Client, this is done automatically for us. The only thing we need to do is check the loading state by returning it from the `useQuery` composable:

📃 **App.vue**

    const { result, loading } = useQuery(ALL_BOOKS_QUERY, () => ({
      search: searchTerm.value,
    }))
    
    return { result, searchTerm, loading }
    

Now we can modify our template to display a “Loading…” message conditionally whenever a query is in progress, and otherwise render the query’s results:

📃 **App.vue**

    <template>
      <div>
        <input type="text" v-model="searchTerm" />
        <p v-if="loading">Loading...</p>
        <template v-else>
          <p v-for="book in result.allBooks" :key="book.id">
            {{ book.title }}
          </p>
        </template>
      </div>
    </template>
    

Let’s look at the rendered result:

You might have noticed that when we type the same search term for the second time, “Loading…” is not displayed. This is because after we fire the same query with the same variable once, the result is stored in the Apollo cache. All subsequent queries with the same variables will receive a result from the cache without calling the API, so there is no loading state in this case.

You can always check cached results in Apollo browser devtools:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F7.1642010026504.jpg?alt=media&token=311a1dd2-b9e5-49cb-9adb-4c0c4a1c191d](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F7.1642010026504.jpg?alt=media&token=311a1dd2-b9e5-49cb-9adb-4c0c4a1c191d)

* * *

Handling query errors
---------------------

Sometimes our queries return an error. It might be a network error, wrong query name or asking for the fields that don’t exist in the schema, a wrong variable type, or something else.

On the frontend side, we need to handle these errors gracefully and show a proper warning to the user.

In the case of Apollo Client and VueApollo, an `error` property is returned from the `useQuery` composable similarly to `loading`:

📃 **App.vue**

    const { result, loading, error } = useQuery(ALL_BOOKS_QUERY, () => ({
      search: searchTerm.value,
    }))
    
    return { result, searchTerm, loading, error }
    

We can display that `error` result conditionally:

📃 **App.vue**

    <template>
      <div>
        <input type="text" v-model="searchTerm" />
        <p v-if="loading">Loading...</p>
        <p v-else-if="error">Something went wrong! Please try again</p>
        <template v-else>
          <p v-for="book in result.allBooks" :key="book.id">
            {{ book.title }}
          </p>
        </template>
      </div>
    </template>
    

Now let’s modify our `allBooks` query to make a failing request:

📃 **allBooks.query.gql**

    query AllBooks($search: String) {
      allBooks(search: $search) {
        test # wrong property, doesn't exist in our GraphQL schema
        id
        title
        rating
      }
    }
    

Now our application displays an error:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F8.1642010028452.jpg?alt=media&token=d4898bd1-cd3d-48d5-874e-c99471d1adf8](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F8.1642010028452.jpg?alt=media&token=d4898bd1-cd3d-48d5-874e-c99471d1adf8)

* * *

In this lesson, we mastered our knowledge of GraphQL queries. Now, we know how to work with variables, handle loading and error states and how the query results are stored in the Apollo cache. In the next lesson, we will continue with some more advanced cases of GraphQL queries.

### Lesson Resources

##### Source Code:

*   [Server Repo](https://github.com/Code-Pop/graphql-server)
    
*   [Client Repo Starting Code](https://github.com/Code-Pop/graphql-client/tree/L4-start)
    
*   [Client Repo Ending Code](https://github.com/Code-Pop/graphql-client/tree/L4-end)
