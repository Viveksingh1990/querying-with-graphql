Real-time updates with subscriptions
====================================

If we want to build a real-time application (for example, a chat) it might be useful to have GraphQL subscriptions in our application. Subscriptions maintain an active connection to your GraphQL server (most commonly via WebSocket). This enables your server to push updates to the subscription’s result over time. In this lesson, we will learn how to handle subscriptions in your Vue application.

* * *

Testing a subscription on GraphQL playground
--------------------------------------------

Let’s start with running a subscription in our GraphQL playground on [`http://localhost:4000/graphql`](http://localhost:4000/graphql) . First, as usual, we check the schema to learn how the subscription should look like:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1645644096515.jpg?alt=media&token=242bedf6-8e01-4f78-a89d-05ae03cade9b](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1645644096515.jpg?alt=media&token=242bedf6-8e01-4f78-a89d-05ae03cade9b)

As we can see, we don’t need any additional parameters to be passed and subscription will return us a new book. It’s not clear from the playground, but this subscription will be triggered every single time a new book is added to the API.

Let’s write our subscription and run it:

    subscription bookSubscription {
      bookSub {
        id
        title
        author
        year
        rating
      }
    }
    

There should be a loading spinner on the right side of the playground:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1645644096516.jpg?alt=media&token=17725134-0666-4e18-9ef0-0a0d2c43461f](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1645644096516.jpg?alt=media&token=17725134-0666-4e18-9ef0-0a0d2c43461f)

Now, let’s open one more tab in our playground and re-create a mutation to add a book there:

    mutation AddBook {
      addBook(input: {
        title: "Subscriptions",
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
    

Now, let’s run the mutation and switch back to the subscription tab:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1645644102850.gif?alt=media&token=471a27bd-c2fb-49bc-84a8-2f27fdc1c693](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1645644102850.gif?alt=media&token=471a27bd-c2fb-49bc-84a8-2f27fdc1c693)

This means that with the mutation changing data on the server, our subscription was triggered and returned us a new book!

Now, let’s start implementing this in our Vue application.

* * *

Setting up Apollo Client for subscriptions
------------------------------------------

Before we can start using a subscription, we need to make a few modifications to our client setup. Namely, we need to create an Apollo link for the websocket connection and pass it to the client constructor.

As a first step, let’s install a `subscriptions-transport-ws` package (it will be required by Apollo as a missing peer dependency):

    yarn add subscriptions-transport-ws
    ## OR
    npm install subscriptions-transport-ws
    

Then, let’s import `WebSocketLink` from Apollo to our `main.js` and create one:

📃 **main.js**

    import { WebSocketLink } from '@apollo/client/link/ws'
    
    const wsLink = new WebSocketLink({
      uri: `ws://localhost:4000/graphql`,
      options: {
        reconnect: true,
      },
    })
    

As you can see, we use the same URL as for the HTTP link, but a different protocol: `ws` instead of `http`. The exact URL for any given case is usually defined on the server side.

Now, we need to _split_ our links to tell the client what link should be use in which case. Namely, for subscriptions we are going to use `wsLink` and for the rest of the operations we want to use `httpLink` . We need to import `split` and `getMainDefinition` helpers from Apollo for this:

📃 **main.js**

    import {
      ApolloClient,
      createHttpLink,
      InMemoryCache,
      split,
    } from '@apollo/client/core'
    import { getMainDefinition } from '@apollo/client/utilities'
    
    const httpLink = createHttpLink({
      uri: 'http://localhost:4000/graphql',
    })
    
    const wsLink = new WebSocketLink({
      uri: `ws://localhost:4000/graphql`,
      options: {
        reconnect: true,
      },
    })
    
    const link = split(
      ({ query }) => {
        const definition = getMainDefinition(query)
        return (
          definition.kind === 'OperationDefinition' &&
          definition.operation === 'subscription'
        )
      },
      wsLink,
      httpLink
    )
    

Now, we only need to replace `link` option in the client constructor:

📃 **main.js**

    const apolloClient = new ApolloClient({
      link,
      cache,
    })
    

* * *

Using the subscription in the application
-----------------------------------------

We are ready to update our list in the Vue application whenever a new book is added on the server!

Let’s start with creating a `newBook.subscription.gql` file and adding our subscription.

💡 Don't forget to utilize the power of fragments!

📃 **graphql/newBook.subscription.gql**

    #import './book.fragment.gql'
    
    subscription bookSubscription {
      bookSub {
        ...BookFragment
      }
    }
    

Let’s now import this subscription to the `App.vue`:

    // App.vue
    
    import BOOK_SUBSCRIPTION from './graphql/newBook.subscription.gql'
    

Before using the subscription, let’s try to think: _what do we want to do when we receive an update from the server about a new book added?_

We have a list of books and most likely we want to update it by appending a new book there. As you remember, the list of books is a result of the GraphQL query, and when we want to update the result of the query with a subscription, we should use the `subscribeToMore` option.

What does `subsribeToMore` do? It listens to the GraphQL subscription and, when we have a new result, can update the current query with this result. Let’s start implementing `subscribeToMore` in our `allBooks` query.

We will start with exposing the `subscribeToMore` hook in the `useQuery` call:

📃 **App.vue**

    const { result, loading, error, subscribeToMore } = useQuery(
      allBooksQuery,
      () => ({
        search: searchTerm.value,
      }),
      () => ({
        debounce: 500,
        // enabled: searchTerm.value.length > 2,
      })
    )
    
    subscribeToMore()
    

After adding `subscribeToMore` call you’ll notice that our application stopped working. When checking the console, we can see the error saying `Cannot read property 'document' of undefined`. This is happening because `subscribeToMore` has a required option `document` where we need to specify the subscription we are listening to. (i.e. `BOOK_SUBSCRIPTION`). Let’s pass it as an argument:

📃 **App.vue**

    subscribeToMore(() => ({
      document: BOOK_SUBSCRIPTION,
    }))
    

This fixes an error but we still need to implement the logic for updating the query. To do so, we need to utilize the `updateQuery` option:

📃 **App.vue**

    subscribeToMore(() => ({
      document: BOOK_SUBSCRIPTION,
      updateQuery(previousResult, newResult) {
        console.log({ previousResult, newResult })
      },
    }))
    

So far, we are not doing anything with the cache. We just want to see what we have in the previous result and what comes with the subscription. Let’s run the mutation on the playground.

As we can see in the console, `previousResult` contains the `allBooks` array and `newResult` has the following structure:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1645644105651.jpg?alt=media&token=0369806c-0a87-46c1-8fee-1fcd1660b7f2](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1645644105651.jpg?alt=media&token=0369806c-0a87-46c1-8fee-1fcd1660b7f2)

So, what we need to do, is:

*   take the previous query result
*   add the content of the `newResult.subscriptionData.data.bookSub` to `previousResult.allBooks`
*   return this from `updateQuery`

📃 **App.vue**

    subscribeToMore(() => ({
      document: BOOK_SUBSCRIPTION,
      updateQuery(previousResult, newResult) {
        const res = {
          allBooks: [
            ...previousResult.allBooks,
            newResult.subscriptionData.data.bookSub,
          ],
        }
        return res
      },
    }))
    

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F6.1645644109656.gif?alt=media&token=f1e79de5-8975-46a7-9195-0ed6055384d9](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F6.1645644109656.gif?alt=media&token=f1e79de5-8975-46a7-9195-0ed6055384d9)

* * *

In this lesson, we learned how to react to the changes on the server side in real time and how to update the application UI using GraphQL subscriptions.

### Lesson Resources

##### Source Code:

*   [Server Repo](https://github.com/Code-Pop/graphql-server)
    
*   [Client Repo Starting Code](https://github.com/Code-Pop/graphql-client/tree/L9-start)
    
*   [Client Repo Ending Code](https://github.com/Code-Pop/graphql-client/tree/L9-end)
