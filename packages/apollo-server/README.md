# 源码分析

## 知识点

### DataSources的使用和解析

一般的使用是通过：

``` js
const server = new ApolloServer({
  typeDefs,
  resolvers,
  // 将dataSources挂载到context上下文
  dataSources: () => ({
    launchAPI: new LaunchAPI(),
    userAPI: new UserAPI({ store }),
  }),
});
```

apollo-server-core会初始化时将传入的dataSources挂载到context.dataSources上，具体见[此处](https://github.com/FunnyLiu/apollo-server/blob/readsource/packages/apollo-server-core/src/requestPipeline.ts#L734)。

从而提供给resolvers消费，比如下面的dataSources参数。

``` js
module.exports = {
  Query: {
    //   分页
    launches: async (_, { pageSize = 20, after }, { dataSources }) => {
      const allLaunches = await dataSources.launchAPI.getAllLaunches();
      // we want these in reverse chronological order
      allLaunches.reverse();
      const launches = paginateResults({
        after,
        pageSize,
        results: allLaunches,
      });
      return {
        launches,
        cursor: launches.length ? launches[launches.length - 1].cursor : null,
        // if the cursor at the end of the paginated results is the same as the
        // last item in _all_ results, then there are no more results after this
        hasMore: launches.length
          ? launches[launches.length - 1].cursor !==
            allLaunches[allLaunches.length - 1].cursor
          : false,
      };
    },
    launch: (_, { id }, { dataSources }) =>
      dataSources.launchAPI.getLaunchById({ launchId: id }),
    me: (_, __, { dataSources }) => dataSources.userAPI.findOrCreateUser(),
  }
};

```

### resolvers和typeDefs等参数的解析

这些实际上是基于graphql-tools这个包来完成的。包源码在[此处](https://github.com/FunnyLiu/graphql-tools/tree/readsource)。

解析的地方在[此处](https://github.com/FunnyLiu/apollo-server/blob/readsource/packages/apollo-server-core/src/ApolloServer.ts#L489)，从而挂载到context上。




### listen

实现参考[此处](https://github.com/FunnyLiu/apollo-server/blob/readsource/packages/apollo-server/src/index.ts#L92)



### 请求查询的流程

首先在入口也就是[此处](https://github.com/FunnyLiu/apollo-server/blob/readsource/packages/apollo-server-express/src/ApolloServer.ts#L194)，增加一个/graphql的接口路由。

经过一系列处理。

最后通过graphql-js的execute方法来完成真正的操作，参考：[此处](https://github.com/FunnyLiu/apollo-server/blob/readsource/packages/apollo-server-core/src/requestPipeline.ts#L447)



# <a href='https://www.apollographql.com/'><img src='https://user-images.githubusercontent.com/841294/53402609-b97a2180-39ba-11e9-8100-812bab86357c.png' height='100' alt='Apollo Server'></a>
## GraphQL Server for Express, Koa, Hapi, Lambda, and more.

[![npm version](https://badge.fury.io/js/apollo-server-core.svg)](https://badge.fury.io/js/apollo-server-core)
[![Build Status](https://circleci.com/gh/apollographql/apollo-server/tree/main.svg?style=svg)](https://circleci.com/gh/apollographql/apollo-server/tree/main)
[![Join the community on Spectrum](https://withspectrum.github.io/badge/badge.svg)](https://spectrum.chat/apollo)
[![Read CHANGELOG](https://img.shields.io/badge/read-changelog-blue)](https://github.com/apollographql/apollo-server/blob/HEAD/CHANGELOG.md)

Apollo Server is a community-maintained open-source GraphQL server. It works with pretty much all Node.js HTTP server frameworks, and we're happy to take PRs to add more! Apollo Server works with any GraphQL schema built with [GraphQL.js](https://github.com/graphql/graphql-js)--or define a schema's type definitions using schema definition language (SDL).

[Read the documentation](https://www.apollographql.com/docs/apollo-server/) for information on getting started and many other use cases and [follow the CHANGELOG](https://github.com/apollographql/apollo-server/blob/HEAD/CHANGELOG.md) for updates.

## Principles

Apollo Server is built with the following principles in mind:

- **By the community, for the community**: Its development is driven by the needs of developers.
- **Simplicity**: By keeping things simple, it is more secure and easier to implement and contribute.
- **Performance**: It is well-tested and production-ready.

Anyone is welcome to contribute to Apollo Server, just read [CONTRIBUTING.md](./CONTRIBUTING.md), take a look at the [roadmap](./ROADMAP.md) and make your first PR!

## Getting started

To get started with Apollo Server:

* Install with `npm install apollo-server-<integration>`
* Write a GraphQL schema
* Use one of the following snippets

There are two ways to install Apollo Server:

* **[Standalone](#installation-standalone)**: For applications that do not require an existing web framework, use the `apollo-server` package.
* **[Integrations](#installation-integrations)**: For applications with a web framework (e.g. `express`, `koa`, `hapi`, etc.), use the appropriate Apollo Server integration package.

For more info, please refer to the [Apollo Server docs](https://www.apollographql.com/docs/apollo-server/v2).

### Installation: Standalone

In a new project, install the `apollo-server` and `graphql` dependencies using:

    npm install apollo-server graphql

Then, create an `index.js` which defines the schema and its functionality (i.e. resolvers):

```js
const { ApolloServer, gql } = require('apollo-server');

// The GraphQL schema
const typeDefs = gql`
  type Query {
    "A simple type for getting started!"
    hello: String
  }
`;

// A map of functions which return data for the schema.
const resolvers = {
  Query: {
    hello: () => 'world',
  },
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
});

server.listen().then(({ url }) => {
  console.log(`🚀 Server ready at ${url}`);
});
```

> Due to its human-readability, we recommend using [schema-definition language (SDL)](https://www.apollographql.com/docs/apollo-server/essentials/schema/#schema-definition-language) to define a GraphQL schema--[a `GraphQLSchema` object from `graphql-js`](https://github.com/graphql/graphql-js/#using-graphqljs) can also be specified instead of `typeDefs` and `resolvers` using the `schema` property:
>
> ```js
> const server = new ApolloServer({
>   schema: ...
> });
> ```

Finally, start the server using `node index.js` and go to the URL returned on the console.

For more details, check out the Apollo Server [Getting Started guide](https://www.apollographql.com/docs/apollo-server/getting-started.html) and the [fullstack tutorial](https://www.apollographql.com/docs/tutorial/introduction.html).

For questions, the [Apollo community on Spectrum.chat](https://spectrum.chat) is a great place to get help.

## Installation: Integrations

While the standalone installation above can be used without making a decision about which web framework to use, the Apollo Server integration packages are paired with specific web frameworks (e.g. Express, Koa, hapi).

The following web frameworks have Apollo Server integrations, and each of these linked integrations has its own installation instructions and examples on its package `README.md`:

- [Express](https://github.com/apollographql/apollo-server/tree/main/packages/apollo-server-express) _(Most popular)_
- [Koa](https://github.com/apollographql/apollo-server/tree/main/packages/apollo-server-koa)
- [Hapi](https://github.com/apollographql/apollo-server/tree/main/packages/apollo-server-hapi)
- [Fastify](https://github.com/apollographql/apollo-server/tree/main/packages/apollo-server-fastify)
- [Amazon Lambda](https://github.com/apollographql/apollo-server/tree/main/packages/apollo-server-lambda)
- [Micro](https://github.com/apollographql/apollo-server/tree/main/packages/apollo-server-micro)
- [Azure Functions](https://github.com/apollographql/apollo-server/tree/main/packages/apollo-server-azure-functions)
- [Google Cloud Functions](https://github.com/apollographql/apollo-server/tree/main/packages/apollo-server-cloud-functions)
- [Cloudflare](https://github.com/apollographql/apollo-server/tree/main/packages/apollo-server-cloudflare) _(Experimental)_

## Context

A request context is available for each request.  When `context` is defined as a function, it will be called on each request and will receive an object containing a `req` property, which represents the request itself.

By returning an object from the `context` function, it will be available as the third positional parameter of the resolvers:

```js
new ApolloServer({
  typeDefs,
  resolvers: {
    Query: {
      books: (parent, args, context, info) => {
        console.log(context.myProperty); // Will be `true`!
        return books;
      },
    }
  },
  context: async ({ req }) => {
    return {
      myProperty: true
    };
  },
})
```

## Documentation

The [Apollo Server documentation](https://apollographql.com/docs/apollo-server/) contains additional details on how to get started with GraphQL and Apollo Server.

The raw Markdown source of the documentation is available within the `docs/` directory of this monorepo--to contribute, please use the _Edit on GitHub_ buttons at the bottom of each page.

## Development

If you wish to develop or contribute to Apollo Server, we suggest the following:

- Fork this repository

- Install the Apollo Server project on your computer

```
git clone https://github.com/[your-user]/apollo-server
cd apollo-server
npm install
cd packages/apollo-server-<integration>/
npm link
```

- Install your local Apollo Server in the other App

```
cd ~/myApp
npm link apollo-server-<integration>
```

## Community

Are you stuck? Want to contribute? Come visit us in the [Apollo community on Spectrum.chat!](https://spectrum.chat/apollo)


## Maintainers

- [@abernix](https://github.com/abernix) (Apollo)
