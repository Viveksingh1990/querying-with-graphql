Fetching data with queries
==========================

Before making our first GraphQL query from a Vue application, it’s helpful to check out the GraphQL playground so we can learn how to write simple queries there first.

Working with the GraphQL playground
-----------------------------------

To run a GraphQL server locally, you will need to clone the [server repository](https://gitlab.com/ntepluhina/vuemastery-graphql-server) for this course and install the dependencies in its root folder by running:

    npm install
    ## OR
    yarn
    

After this, you’ll run the `apollo` script:

    npm run apollo
    ## OR
    yarn apollo
    

As a result, you should have a GraphQL server running on [http://localhost:4000/graphql](http://localhost:4000/graphql)

* * *

Let’s click the `Docs` tab on the right and explore a bit. You can see that the first query in the list is `allBooks`, which returns an array of `Book`s:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1638987083509.jpg?alt=media&token=cc907aaa-652a-40f8-8e2c-566b0230962b](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1638987083509.jpg?alt=media&token=cc907aaa-652a-40f8-8e2c-566b0230962b)

Here you can see possible fields you can get when fetching a list of books.

With this in mind, let’s write our first query in the playground, where we get the `id` of our books like so:

    query AllBooks {
      allBooks {
        id
      }
    }
    

(The query name `AllBooks` here is optional and can be anything; however, it’s highly recommended to give your queries names since this will help with debugging them later.)

Now that our query is ready, let’s click on the **Run** button and check the response.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1638987083510.gif?alt=media&token=a4c23b7c-f9aa-4685-86ae-8489a5180be8](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1638987083510.gif?alt=media&token=a4c23b7c-f9aa-4685-86ae-8489a5180be8)

So far so good! We got an array full of one object for each book, containing its `id`. Now let’s add more fields to the query to get more info about these books.

    query AllBooks {
      allBooks {
        id
        title
        rating
      }
    }
    

You can see how the response structure has changed to reflect the request changes.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1638987088961.jpg?alt=media&token=a3c60974-3df1-4ad8-8ae6-61a5933b27d8](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1638987088961.jpg?alt=media&token=a3c60974-3df1-4ad8-8ae6-61a5933b27d8)

Now that we have a bit of practice with writing queries in the playground, let’s start implementing them on the frontend side and add the very first query to our Vue application.

* * *

Installing dependencies
-----------------------

💡 This course assumes you are using a webpack build like an application created with Vue CLI

Before we get started, we need to install some packages from the terminal:

    npm install graphql@15 graphql-tag @apollo/client @vue/apollo-composable
    ## OR
    yarn add graphql@15 graphql-tag @apollo/client @vue/apollo-composable
    

Let’s take a look at these packages:

*   `graphql` - the JavaScript reference implementation for the GraphQL query language.
*   `graphql-tag` - a template literal that will help us to ‘parse’ our GraphQL query strings. This package also contains a webpack loader (we will take a look over this one later in the lesson)
*   `@apollo/client` - Apollo client for GraphQL that will help us send requests to the GraphQL server, track loading state, and cache the results
*   `@vue/apollo-composable` - a Vue integration for Apollo client that uses Vue Composition API

Then, in our application, let’s open the `main.js` file and create an ApolloClient instance. This instance will serve to manage both local and remote data in our application. With it, we will be able to fetch data from the GraphQL API, change the data on the server, and update the local cache with new data.

**main.js**

    import {
      ApolloClient,
      createHttpLink,
      InMemoryCache,
    } from "@apollo/client/core";
    
    const httpLink = createHttpLink({
      uri: 'http://localhost:4000/graphql',
    })
    
    const cache = new InMemoryCache()
    
    const apolloClient = new ApolloClient({
      link: httpLink,
      cache,
    })
    

With `createHttpLink`, we create a link to our GraphQL API. In our case, the server is running on [http://localhost:4000/graphql](http://localhost:4000/graphql).

`cache` is the Apollo Client cache, where all the queried data will be stored. We’ll look deeper into this later.

The Apollo Client instance already allows us to perform a query with its `watchQuery` method, so let’s copy our query from the playground and save it as a constant within **main.js**. For this, we will need a `graphql-tag`, which is a utility to parse GraphQL queries.

**main.js**

    import gql from 'graphql-tag'
    
    const ALL_BOOKS_QUERY = gql`
      query AllBooks {
        allBooks {
          id
          title
          rating
        }
      }
    `
    

Now, we’ll call the API and log the result in the browser console:

**main.js**

    const apolloClient = new ApolloClient({
      link: httpLink,
      cache,
    })
    
    const ALL_BOOKS_QUERY = gql`
      query AllBooks {
        allBooks {
          id
          title
          rating
        }
      }
    `
    
    apolloClient
      .query({
        query: ALL_BOOKS_QUERY,
      })
      .then(res => {
        console.log(res)
      })
    

Here, we are making a simple GraphQL query like we would call a REST API. We make a network request to our GraphQL server, and when it’s resolved, we show the response in the browser console.

At this moment, we should see a result with seven books in the list:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.1638987092330.jpg?alt=media&token=26db9ec7-fbcc-4f33-8d4f-08ac01e77293](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.1638987092330.jpg?alt=media&token=26db9ec7-fbcc-4f33-8d4f-08ac01e77293)

With `apolloClient.query` you can perform GraphQL calls from _outside Vue components_: you might need this for example if you decide to keep the local store in Vuex. But what if we want to make an in-component call and connect its result directly with Vue reactive properties?

* * *

Adding Apollo to the Vue app
----------------------------

In the **main.js** file, we need to import `DefaultApolloClient` from VueApollo. This is a VueApollo helper required for providing the Apollo client instance to the Vue application.

**main.js**

    import { DefaultApolloClient } from '@vue/apollo-composable'
    

Next, we need to import a `provide` method from Vue and modify our `createApp` call to inject the client in our application:

**main.js**

    import { createApp, h, provide } from 'vue'
    
    const apolloClient = new ApolloClient({
      link: httpLink,
      cache,
    })
    
    const app = createApp({
      setup() {
        provide(DefaultApolloClient, apolloClient)
      },
      render: () => h(App),
    })
    

Now we are ready to call a query from the Vue component! With this, we can connect the result of the query to the Vue component’s reactive property, and render this property in the template. Let’s copy our query constant to the `App.vue` component and import the `useQuery` method from VueApollo there:

**App.vue**

    import { useQuery } from '@vue/apollo-composable'
    import gql from 'graphql-tag'
    
    const ALL_BOOKS_QUERY = gql`
      query AllBooks {
        allBooks {
          id
          title
          rating
        }
      }
    `
    

After this, we should add a `setup()` option to our component instance. This is a component property added in Vue 3. It’s executed before the component is created, once the props are resolved, and serves as the entry point for _composables_. `useQuery` is one of such composables exposed by the VueApollo library. Under the hood, it performs a GraphQL query, handles loading and error state, and returns them back to the component.

**App.vue**

    import { useQuery } from '@vue/apollo-composable'
    import gql from 'graphql-tag'
    
    const ALL_BOOKS_QUERY = gql`
      query AllBooks {
        allBooks {
          id
          title
          rating
        }
      }
    `
    
    export default {
      name: 'App',
      setup() {},
    }
    

Now, within `setup()`, let’s call `useQuery` and show the returned `result` in the console:

    setup() {
      const { result } = useQuery(ALL_BOOKS_QUERY)
    
      console.log(result)
    
      return { result }
    },
    

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1638987095399.jpg?alt=media&token=ccf3ae4d-d4db-4d49-94b7-166f6497f6fa](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1638987095399.jpg?alt=media&token=ccf3ae4d-d4db-4d49-94b7-166f6497f6fa)

As you can see, what got returned is a [ref](https://v3.vuejs.org/api/refs-api.html#refs), which means it is a reactive object with the value property. If we were to access it from JavaScript, we’d need to write `result.value.allBooks`, but when we render it in the component template, it’s _automatically unwrapped._

This means that in the template we don’t need to access `.value`:

    <!-- App.vue -->
    
    <template>
      <div>
        <p v-for="book in result.allBooks" :key="book.id">
          {{ book.title }}
        </p>
      </div>
    </template>
    

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F6.1638987098258.jpg?alt=media&token=86345b49-1850-44ba-a5a1-83b2e0061992](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F6.1638987098258.jpg?alt=media&token=86345b49-1850-44ba-a5a1-83b2e0061992)

* * *

* * *

Conclusion
----------

In this lesson, we learned how to make simple queries to fetch data from the GraphQL API, and how to work with this data in Vue components. In the next few lessons we will work on setting up some tools to improve the developer experience and then we come back to learn more advanced GraphQL queries

### Lesson Resources

##### Source Code:

*   [Server Repo](https://github.com/Code-Pop/graphql-server)
    
*   [Client Repo Starting Code](https://github.com/Code-Pop/graphql-client/tree/L2-start)
    
*   [Client Repo Ending Code](https://github.com/Code-Pop/graphql-client/tree/L2-end)
