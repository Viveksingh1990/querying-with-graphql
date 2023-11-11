Advanced queries
================

In the previous lesson, we created a typeahead search by passing a parameter to the GraphQL query. However, it has a few flaws, which we will fix in this lesson using some advanced options of Apollo query.

* * *

Debouncing a query
------------------

With our current implementation, we send a query on every new typed character. But imagine if we decided to type a really long book title! This can result in many unnecessary queries while we only need one.

To limit the number of network requests, we can use a method called _debouncing_: where we limit the rate at which the query is fired. For example, we could add a 0.5 delay after the user stops typing before sending a query.

Luckily with Apollo we don’t need any external libraries to implement debouncing for our queries. We can pass a delay (in milliseconds) to `useQuery` function as a property of the third argument:

📃 **App.vue**

    // App.vue
    
    const { result, loading, error } = useQuery(
      // query string
      ALL_BOOKS_QUERY,
      // query variables
      () => ({ 
        search: searchTerm.value,
      }),
      // query options
      () => ({
        debounce: 500,
      })
    )
    

With this simple option, we only send out the query when the user stops typing:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1642614381183.gif?alt=media&token=b05a34c5-c5ce-47e6-a206-49ad356f79fa](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1642614381183.gif?alt=media&token=b05a34c5-c5ce-47e6-a206-49ad356f79fa)

* * *

Disabling a query
-----------------

Another best practice when it comes to search (especially if the list if very big) is to start searching only after the search term has at least 2-3 characters. Let’s implement this in our application.

In the `useQuery` method options, we can specify the condition to _enable_ the query. This means the the query will only be fired when the condition returns `true`. In our case, we only want to run the search when the user has typed more than two characters in the input field:

📃 **App.vue**

    setup() {
      const searchTerm = ref('')
      const { result, loading, error } = useQuery(
        ALL_BOOKS_QUERY,
        () => ({
          search: searchTerm.value,
        }),
        () => ({
          debounce: 500,
          enabled: searchTerm.value.length > 2,
        })
      )
    
      return { result, searchTerm, loading, error }
    },
    

Surprisingly, when adding this condition, our application is broken…

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1642614381184.jpg?alt=media&token=c2230651-a543-49fe-8b3f-c6a0231eed5b](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1642614381184.jpg?alt=media&token=c2230651-a543-49fe-8b3f-c6a0231eed5b)

This happened because we tried to access a property `allBooks` of the query result but the query is not fired, so `result` is undefined.

We will handle this properly in a few minutes. For now we can change a condition on the template to only display the list when we have a `result`:

📃 **App.vue**

    <template v-else-if="result">
      <p v-for="book in result.allBooks" :key="book.id">
        {{ book.title }}
      </p>
    </template>
    

Now, the page will be empty until we type 3 characters:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1642614387393.gif?alt=media&token=8bb67e48-2c83-4e5c-b893-08844e59e1e0](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1642614387393.gif?alt=media&token=8bb67e48-2c83-4e5c-b893-08844e59e1e0)

* * *

Extracting the query result
---------------------------

As we just saw, sometimes it can be handy to have a default value for the query result to prevent cases where it’s `undefined`. Also, it would be nice to extract a nested property of it—we don’t really need a `result`, we need only `result.allBooks` to render in our template.

To handle such cases, we can use VueApollo’s `useResult` composable.

`useResult` is designed to:

*   provide a default value in the cases when `result` is undefined
*   pick one object from the query result when `result` is not undefined

Let’s retrieve the `allBooks` property and save it into a new `books` const:

📃 **App.vue**

    import { useQuery, useResult } from '@vue/apollo-composable'
    
    export default {
      setup() {
        const searchTerm = ref('')
        const { result, loading, error } = useQuery(
          ALL_BOOKS_QUERY,
          () => ({
            search: searchTerm.value,
          }),
          () => ({
            debounce: 500,
            enabled: searchTerm.value.length > 2,
          })
        )
    
        const books = useResult(result, [], data => data.allBooks)
    
        return { books, searchTerm, loading, error }
      },
    }
    

So what are we passing into `useResult`?

The first argument here is the result we got from the query.

The second one is a _default value_ for `books`. In case there is no `result`, we want it to be an empty array so when the query is not called, we will be iterating over an empty list. This will allow us to remove the `v-else-if="result"` check in the template.

Finally, the third argument is a function that defines what data we want to extract from the query response. In our case, we want to get the `allBooks` array.

Now we just need to remove `result` from the `setup()` return object and return `books` instead.

    return { books, searchTerm, loading, error }
    

Also, we need to make changes to the component’s template:

📃 **App.vue**

    <div>
      <input type="text" v-model="searchTerm" />
      <p v-if="loading">Loading...</p>
      <p v-else-if="error">Something went wrong! Please try again</p>
      <template v-else>
        <p v-for="book in books" :key="book.id">
          {{ book.title }}
        </p>
      </template>
    </div>
    

A bit cleaner, right?

* * *

Wrapping up
-----------

In this lesson, we learned a few techniques we can use to craft our queries. Now we know how to add debouncing to limit the query calls rate, and we learned how to enable queries only when a certain condition is met. Finally, we did a nice small refactoring to the existing code by picking `allBooks` from the query result and providing a default value for it. In the next lesson, we’re going to start modifying data we store on the server - we will be learning GraphQL mutations.

### Lesson Resources

##### Source Code:

*   [Server Repo](https://github.com/Code-Pop/graphql-server)
    
*   [Client Repo Starting Code](https://github.com/Code-Pop/graphql-client/tree/L5-start)
    
*   [Client Repo Ending Code](https://github.com/Code-Pop/graphql-client/tree/L5-end)
