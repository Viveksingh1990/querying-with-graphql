Modifying local state with Apollo Client
========================================

Next step… Let’s try to add something to the list of favorites.

* * *

Adding a new book to the list
-----------------------------

Modifying the local data in the cache is not really different from modifying the data on the server—both are done with GraphQL mutations. For the server data, the mutation is defined on our GraphQL API. For the local modifications, we need to add the mutation first to our local schema.

Let’s define it on out **typedefs.gql** file:

📃 **typedefs.gql**

    extend type Mutation {
      addBookToFavorites(book: Book!): [Book]
    }
    

What is happening here? We defined a few important things:

*   `addBookToFavorites` - this is a mutation name. We can track it in Apollo browser devtools with this name, and we will use it later to write a resolver.
*   `(book: Book!)` - we need to define the variable that the mutation will accept. We plan to add books from our main list to the list of favorites, and all books in our main list have a type of `Book`
*   `[Book]` - finally, we want to return an array of books here

Now that we know how the mutation should look, we can create an **addBookToFavorites.mutation.gql** file and write the mutation there:

**addBookToFavorites.mutation.gql**

    #import './book.fragment.gql'
    
    mutation addBookToFavorites($book: Book!) {
      addBookToFavorites(book: $book) @client {
        ...BookFragment
      }
    }
    

Again, we use `@client` directive to tell our Apollo Client not to send this mutation to the remote GraphQL server, but only operate on the cache level.

Let’s try to call this mutation from our application and check it in the devtools!

In the `App.vue` we need to add `useMutation` to our imports:

📃 **App.vue**

    import { useQuery, useResult, useMutation } from '@vue/apollo-composable'
    import ADD_BOOK_TO_FAVORITES_MUTATION from './graphql/addBookToFavorites.mutation.mutation.gql'
    

At the end of the `setup` function, let’s add a mutation:

📃 **App.vue**

    setup() {
    	/* code from previous lessons */
    
    	const { mutate: addBookToFavorites } = useMutation(
    	  ADD_BOOK_TO_FAVORITES_MUTATION
    	)
    
    	return {
    	  books,
    	  searchTerm,
    	  loading,
    	  error,
    	  activeBook,
    	  showNewBookForm,
    	  favBooksResult,
    	  addBookToFavorites,
    	}
    }
    

Finally, we need to add the button to every book in the list to add it to favorites:

📃 **App.vue**

    <div class="list">
      <h3>All Books</h3>
      <p v-for="book in books" :key="book.id">
        {{ book.title }} - {{ book.rating }}
        <button @click="activeBook = book">Edit rating</button>
    		<!-- New button here -->
        <button @click="addBookToFavorites({ book })">
          Add to Favorites
        </button>
      </p>
    </div>
    

As you can see, when we call the mutation, we are passing an object of variables. Our mutation requires one variable called `book` and we pass a book from the `books` array when clicking a button.

Let’s check how it looks in our application’s interface:

![mutation.gif](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.gif?alt=media&token=55b6226f-13b6-4f4e-8c46-3f22b6d16d52)

Awesome! The mutation is called correctly, as we can see from the devtools. The only problem is that it does not update the favorite books list. Why? Because our client can see the mutation is called but does not have any instructions about what to do _when_ it’s called.

Such instructions are called _resolvers_, and they should be written as an object with the fields like `Query`, `Mutation`. Every field should have nested fields named exactly like how we name the query of the mutation.

We didn’t need a resolver for our `favoriteBooks` query because it was a straightforward cache-reading operation. Our new mutation is more complicated and therefor requires a resolver.

* * *

Writing a resolver for addBookToFavorites mutation
--------------------------------------------------

Let’s open our `main.js` file and start with creating a new constant called `resolvers`

📃 **main.js**

    const resolvers = {}
    

Now, we need to add our mutation there:

📃 **main.js**

    const resolvers = {
      Mutation: {
        addBookToFavorites: () => {},
    }
    

So far, it’s just an empty callback. It accepts three arguments: parent (we won’t need this in our resolver so we’ll replace it with underscore, variables that are passed to the mutation when it’s called, and a context. `cache` is one of the context’s properties, and we will need it to read and write the correct query:

📃 **main.js**

    const resolvers = {
      Mutation: {
        addBookToFavorites: (_, { book }, { cache }) => {},
      },
    }
    

Our mutation should modify the `favoriteBooks` array in the cache, so let’s retrieve this array reading the `favoriteBooks`query first:

📃 **main.js**

    import FAVORITE_BOOKS_QUERY from './graphql/favoriteBooks.query.gql'
    
    const resolvers = {
      Mutation: {
        addBookToFavorites: (_, { book }, { cache }) => {
          const data = cache.readQuery({ query: FAVORITE_BOOKS_QUERY })
        },
      },
    }
    

Now, we need to create a new object where `favoriteBooks` contains the previous list and a new book:

📃 **main.js**

    const resolvers = {
      Mutation: {
        addBookToFavorites: (_, { book }, { cache }) => {
          const data = cache.readQuery({ query: FAVORITE_BOOKS_QUERY })
          const newData = {
            favoriteBooks: [...data.favoriteBooks, book],
          }
        },
      },
    }
    

Finally, we need to write this new data back to the cache and return an array of books:

📃 **main.js**

    const resolvers = {
      Mutation: {
        addBookToFavorites: (_, { book }, { cache }) => {
          const data = cache.readQuery({ query: FAVORITE_BOOKS_QUERY })
          const newData = {
            favoriteBooks: [...data.favoriteBooks, book],
          }
          cache.writeQuery({ query: FAVORITE_BOOKS_QUERY, data: newData })
          return newData.favoriteBooks
        },
      },
    }
    

Awesome! The last step we need to take is passing resolvers to Apollo Client constructor:

📃 **main.js**

    const resolvers = {
      Mutation: {
        addBookToFavorites: (_, { book }, { cache }) => {
          const data = cache.readQuery({ query: FAVORITE_BOOKS_QUERY })
          const newData = {
            favoriteBooks: [...data.favoriteBooks, book],
          }
          cache.writeQuery({ query: FAVORITE_BOOKS_QUERY, data: newData })
          return newData.favoriteBooks
        },
      },
    }
    
    const apolloClient = new ApolloClient({
      link,
      cache,
      typeDefs,
      resolvers,
    })
    

Now we can see that the mutation adds the book to the list of favorites:

![result.gif](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1648143879940.gif?alt=media&token=817c0ff5-77f5-4e69-af1f-a66c75b65f4c)

* * *

We did it!
----------

Great job following along. We’ve successfully learned how to use Apollo Client cache as a replacement for your global state and how to work with reading and writing data to it.

### Lesson Resources

##### Source Code:

*   [Server Repo](https://github.com/Code-Pop/graphql-server)
    
*   [Client Repo Starting Code](https://github.com/Code-Pop/graphql-client/tree/L11-start)
    
*   [Client Repo Ending Code](https://github.com/Code-Pop/graphql-client/tree/L11-end)
