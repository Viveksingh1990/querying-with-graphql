Setting up local state with Apollo Client
=========================================

As we saw in previous lessons, Apollo Client caches the data fetched from the API and makes this data available across the application, just like any global state manager. This means Apollo Client is also capable of storing a local application state, even if it was not fetched from the server. In this lesson, we will take a look at local queries, cache updates, and cache invalidation.

* * *

Adding local state to the application
-------------------------------------

Let’s imagine that we want to create a list of a user’s favorite books. This list should be client-side, and we don’t plan to store these books on a server. Also, this list should be available _globally_ in the application— for example, if we want to add different pages. Usually, Vue applications use [Vuex](https://www.vuemastery.com/courses/vuex-fundamentals/vuex4-intro-to-vuex) for the global state management. However, in our case, adding Vuex would mean creating two sources of truth since we already store all the data returned from the server in the Apollo local cache. So is it possible to store _local_ state in the Apollo cache as well? Yes it is!

We can start by defining a local GraphQL schema. Keep in mind that this step is optional but highly recommended. It will help you with tooling such as autocompleting and validating queries in the IDE plugin, generating types if you are using TypeScript in your application, etc.

* * *

Defining local schema
---------------------

As you remember, everything on the GraphQL server is defined in the schema on this server. The schema allows us to see the types, queries, and mutations available in the current API. But what if we want to have some data available only on the client side? A good practice in this case is to define a _local-side_ schema and to let Apollo Client stitch the two together. This way, you will be able to check locally defined types with your IDE plugin for autocompletion.

Let’s create a file `typeDefs.gql` in the `graphql` folder and add a new query there:

📃 **graphql/typeDefs.gql**

    extend type Query {
      favoriteBooks: [Book!]
    }
    

With the code above, we want to tell the Apollo Client that we will have _one more_ query in addition to those defined in GraphQL. This query will return us the list of the user’s favorite books. We also define the type of the returned array. It should be `Book` because we plan to add books from the `allBooks` query to the list of favorites.

Now, we need to pass this schema to Apollo Client. Let’s import it to the **main.js** file and pass it to the client options.

📃 **main.js**

    import typeDefs from './graphql/typedefs.gql'
    
    /* ... */
    
    const apolloClient = new ApolloClient({
      link,
      cache,
      typeDefs,
    })
    

Now, when writing a query, we should have no errors thrown by Apollo IDE plugin.

* * *

Writing a local query
---------------------

How do we usually retrieve data from the Vuex store when we want to use that data in the Vue component? We either access `this.$store.state` or use helpers like `mapState` and `mapAction`.

To get data from the local cache with Apollo Client, we do the same thing we do when retrieving data from the server: use a GraphQL query. As you remember from previous lessons, when we have a query, Apollo Client will first check the local cache for data; if it’s not there, it will send a request to the GraphQL server.

With the _local_ query we don’t want that second part to happen, and we will use the `@client` directive for this. When added to the GraphQL query, the `@client` directive makes Apollo Client only search for the required data in the cache. If there’s nothing in this cache, the client won’t reach out to the server to fetch anything.

Let’s compose our first local query with this directive.

📃 **src/graphql/favoriteBooks.query.gql**

    #import './book.fragment.gql'
    
    query favoriteBooks {
      favoriteBooks @client {
        ...BookFragment
      }
    }
    

As you can see, we reused the same `BookFragment` here to keep the data structure consistent across the application.

Now, let’s use this query in our `App.vue` component to retrieve a list of favourite books:

📃 **App.vue**

    // App.vue
    
    import FAVORITE_BOOKS_QUERY from './graphql/favoriteBooks.query.gql'
    
    export default {
      setup() {
        /* Code from previous lessons */
    
        const { result: favBooksResult } = useQuery(FAVORITE_BOOKS_QUERY)
        console.log(favBooksResult.value)
      },
    }
    

This code will work, but in the console we’ll see `undefined`. Why?

Well, at the moment of the query, there are _no_ favorite books in our Apollo cache. Not even an empty array. To add them there, we need to _add the initial state_ to the cache—just like we would define the initial state in Vuex.

* * *

Setting up initial state in the cache
-------------------------------------

Remember, all the data stored in Apollo cache is structured as GraphQL queries. To add the initial data for our `favoriteBook` query, we need to write it down to the cache when we create an Apollo client instance, so when we run this query on the component, the data would already be in the cache. Let’s open the **main.js** file and find the place where we’re instantiating the cache:

📃 **main.js**

    const cache = new InMemoryCache()
    

Now, we need to import our `favoriteBooks` query into **main.js** and write the initial data to it. Let’s start with an empty array:

📃 **main.js**

    import FAVORITE_BOOKS_QUERY from './graphql/favoriteBooks.query.gql'
    
    /* ... */
    
    const cache = new InMemoryCache()
    
    cache.writeQuery({
      query: FAVORITE_BOOKS_QUERY,
      data: {
        favoriteBooks: [],
      },
    })
    

Now let’s check the browser console when running our application:

![Screenshot 2021-08-14 at 11.46.26.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1646865472901.jpg?alt=media&token=0e08829a-557f-4db2-805d-3d4443cc4235)

Our `favBookResult` on **App.vue** now contains a `favoriteBooks` and it’s an empty array!

Let’s double check with Apollo devtools:

![Screenshot 2021-08-14 at 11.48.07.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1646865472902.jpg?alt=media&token=2c8649f8-dd49-43de-95b2-37e88f11e707)

As you can see in the screenshot above, the `favoriteBooks` are now a part of Apollo cache.

What if we want to have an initial value in our `favoriteBooks`? First of all, we would need a `Book` object to match the query structure. Let’s check what fields `BookFragment` contains:

    fragment BookFragment on Book {
      id
      title
      rating
    }
    

So, our book should contain three these fields. Let’s go forward and add it to the `writeQuery` call:

📃 **main.js**

    cache.writeQuery({
      query: FAVORITE_BOOKS_QUERY,
      data: {
        favoriteBooks: [
          {
            title: 'My Book',
            id: 123,
            rating: 5,
          },
        ],
      },
    })
    

Looks like we added all the fields correctly, but when we check the console, something is clearly off:

![Screenshot 2021-08-14 at 11.56.36.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1646865477902.jpg?alt=media&token=9a2d8b33-fba9-4b37-bc6e-458940e4bf30)

Why is the array empty?

The problem is we did not define the `__typename` for the object, so this object was not recognized by Apollo as a `Book` type. Therefor, it was not written to the cache correctly.

Let’s add the `__typename` here:

📃 **main.js**

    cache.writeQuery({
      query: FAVORITE_BOOKS_QUERY,
      data: {
        favoriteBooks: [
          {
            __typename: 'Book',
            title: 'My Book',
            id: 123,
            rating: 5,
          },
        ],
      },
    })
    

Now we can see the correct result in the console:

![Screenshot 2021-08-14 at 12.01.35.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.1646865480869.jpg?alt=media&token=771a3618-3ec3-43e2-8ba3-fd74648384f2)

* * *

Rendering the list of favorite books
------------------------------------

Now, when we have a list of favorite books, let’s render them within our application. We will start with modifying our template a bit to have two columns: one for all available books and another for favorite books.

Let’s replace this part of the template:

📃 **App.vue**

    <template v-else>
      <p v-for="book in books" :key="book.id">
        {{ book.title }} - {{ book.rating }}
        <button @click="activeBook = book">Edit rating</button>
      </p>
    </template>
    

With this:

📃 **App.vue**

    <template v-else>
      <section class="list-wrapper">
        <div class="list">
          <h3>All Books</h3>
          <p v-for="book in books" :key="book.id">
            {{ book.title }} - {{ book.rating }}
            <button @click="activeBook = book">Edit rating</button>
          </p>
        </div>
        <div class="list">
          <h3>Favorite Books</h3>
        </div>
      </section>
    </template>
    

Also, we need to extend the `style` section:

    .list-wrapper {
      display: flex;
      margin: 0 auto;
      max-width: 960px;
    }
    
    .list {
      width: 50%;
    }
    

Now, our page should look like this:

![Untitled](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1646865483038.jpg?alt=media&token=72186591-7b08-46c1-9db5-33d1248ee19a)

Now we’ll render an actual list of the favorite books in the right column. Let’s modify the template to iterate over `favBooksResult.favoriteBooks` array:

📃 **App.vue**

    <div class="list">
      <h3>Favorite Books</h3>
      <p v-for="book in favBooksResult.favoriteBooks" :key="book.id">
        {{ book.title }}
      </p>
    </div>
    

We should now see the list of favorite books rendered on the page:

![Untitled](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F6.opt.1646865485303.jpg?alt=media&token=44c03b1b-89af-4a55-b1b7-0e2360048045)

Next step… Let’s try to add something to the list of favorites. See you in the next lesson!

### Lesson Resources

##### Source Code:

*   [Server Repo](https://github.com/Code-Pop/graphql-server)
    
*   [Client Repo Starting Code](https://github.com/Code-Pop/graphql-client/tree/L10-start)
    
*   [Client Repo Ending Code](https://github.com/Code-Pop/graphql-client/tree/L10-end)
