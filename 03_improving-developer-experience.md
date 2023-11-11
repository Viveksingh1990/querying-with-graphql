Improving Developer Experience
==============================

Now that we’ve made progress getting GraphQL up and running, it’s a good time to pause to make some optimizations for our developer experience before we go deeper.

* * *

Loading .gql files
------------------

Creating a string-based constant for every single query is not very convenient. For strings, we don’t have proper autocomplete. More to say, having multiple string queries in Vue component will crowd it and make reusing the same query harder. We are going to put our queries in external `.gql` files that can be imported directly into any component.

The way to set this up depends on whether you’re using Vite or Vue CLI.

We are going to start with Vite first.

For Vite apps
-------------

First, we need to install a rollup plugin:

    npm install @rollup/plugin-graphql@1
    

Since this rollup plugin is compatible with Vite, we can use it as a Vite plugin.

To use this new plugin, we have to go to `vite.config.js`. Import the plugin and use it inside the `plugins` array:

📃 **vite.config.js**

    import { defineConfig } from 'vite'
    import vue from '@vitejs/plugin-vue'
    import graphql from '@rollup/plugin-graphql' // ADD
    
    export default defineConfig({
      plugins: [
        vue(), 
        graphql() // ADD
      ]
    })
    

And we are done setting up for Vite.

For Vue CLI apps
----------------

Now for Vue CLI, we don’t have to install any additional plugin.

We need to create a `vue.config.js` file in the root of the project, and add a proper webpack configuration there:

📃 **vue.config.js**

    module.exports = {
      chainWebpack: config => {
        config.module
          .rule('graphql')
          .test(/\.(graphql|gql)$/)
          .use('graphql-tag/loader')
          .loader('graphql-tag/loader')
          .end()
      },
    }
    

This `graphql-loader` comes from the `graphql-tag` package we installed in the previous lesson.

That’s it for the Vue CLI setup.

Now that we’ve seen how to set up the config for both Vite and Vue CLI, I’ll switch back to using the original Vite app for the rest of the lesson. You can follow along even if you’re using Vue CLI.

* * *

Extracting the query to its own file
------------------------------------

Our app is now capable of recognizing import files with the `gql` extension.

Now we can create a new folder called `graphql` and add a file inside called `allBooks.query.gql`, which contains our query:

📃 **graphql/allBooks.query.gql**

    query AllBooks {
      allBooks {
        id
        title
        rating
      }
    }
    

Now in the component, we can remove importing `gql` and the query constant, and instead import the query directly:

📃 **App.vue**

    import { useQuery } from '@vue/apollo-composable'
    import ALL_BOOKS_QUERY from './graphql/allBooks.query.gql'
    
    export default {
      name: 'App',
      setup() {
        const { result } = useQuery(ALL_BOOKS_QUERY)
    
        return { result }
      },
    }
    

💡 It's a useful naming convention to have the query/mutation name as a file name, a type as a first extension, and \`.gql\` as a type

With this knowledge, we can move all the queries to dedicated `.gql` files so they can be easily found in the project structure.

* * *

Code Highlighting and Autocompletion
------------------------------------

So far our code looks very plain without any code highlighting. Also, it would be nice to have some linting and autocomplete on GraphQL files. Using GraphQL schema allows us to do this with ease: any data you can fetch from the GraphQL server is _typed_. This means that for any piece of data there is a defined set of fields, and they can be validated against the schema.

There are a few IDE plugins that can validate GraphQL queries against the schema and enable linting and autocomplete. To improve the developer experience a bit, we can install an IDE plugin: [Apollo GraphQL for Visual Studio code](https://marketplace.visualstudio.com/items?itemName=apollographql.vscode-apollo) or [JS GraphQL plugin for WebStorm](https://plugins.jetbrains.com/plugin/8097-js-graphql/).

As soon as the extension is installed, you should then see your GraphQL code is highlighted.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1640889448261.jpg?alt=media&token=ae551337-c8b6-4b6f-bf31-bec3d1f84e7a](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1640889448261.jpg?alt=media&token=ae551337-c8b6-4b6f-bf31-bec3d1f84e7a)

Now let’s use the full power of the plugin by connecting it to our GraphQL schema! In order to do this, we need to add one more file to the root of the project: `apollo.config.js` for VS Code:

📃 **apollo.config.js**

    module.exports = {
      client: {
        service: {
          name: 'my-graphql-app',
          // URL to the GraphQL API
          url: 'http://localhost:4000/graphql',
        },
        // Files processed by the extension
        includes: ['src/**/*.gql'],
      },
    }
    

Or `.graphqlconfig` for WebStorm:

📃 **.graphqlconfig**

    {
      "name": "My GraphQL Schema",
      "schemaPath": "schema.graphql",
      "extensions": {
        "endpoints": {
          "My GraphQL Endpoint": {
            "url": "http://localhost:3000/api/graphql",
            "introspect": true
          }
        }
      }
    }
    

After restarting your IDE, you should now have GraphQL file validation enabled:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1640889448262.jpg?alt=media&token=1c8979fc-fb64-4563-a9e0-7f2700af1a74](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1640889448262.jpg?alt=media&token=1c8979fc-fb64-4563-a9e0-7f2700af1a74)

* * *

Browser Developer Tools for Easier Debugging
--------------------------------------------

As a final step to improve developer experience, we can install Apollo DevTools (currently available for [Chrome](https://chrome.google.com/webstore/detail/apollo-client-devtools/jdkknkkbebbapilgoeccciglkfbmbnfm?hl=en) and [Firefox](https://addons.mozilla.org/en-US/firefox/addon/apollo-developer-tools/)). After the installation. when running an application in the browser, we should have one more tab in the browser devtools:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1640889453418.jpg?alt=media&token=4fdbfab3-8da1-40de-ba6f-504905765637](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1640889453418.jpg?alt=media&token=4fdbfab3-8da1-40de-ba6f-504905765637)

Here we have a GraphQL playground tab where we can check the schema and run queries, get a list of the called queries and mutations, and also the cache\*.\* The cache is very important when we work with Apollo client because, by default, the result of every query is _cached._ When we send the query subsequently, we are not making a network request, instead we retrieve the data from the Apollo cache. We will look deeper into Apollo caching later

* * *

Ready to move on
----------------

With these tools, we now have a developer-friendly environment that helps us to prevent errors with wrong query fields and makes the debugging process easier. We’re now ready to return to developing with GraphQL in a smoother way.

### Lesson Resources

##### Source Code:

*   [Server Repo](https://github.com/Code-Pop/graphql-server)
    
*   [Client Repo Starting Code](https://github.com/Code-Pop/graphql-client/tree/L3-start)
    
*   [Client Repo Ending Code](https://github.com/Code-Pop/graphql-client/tree/L3-end)
