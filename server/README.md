# Catstronauts - server

The starting point of the `server` code for Odyssey Lift-off I course.

## Getting Started

To get started with our schema, we'll need a couple packages first: `@apollo/server`, `graphql` and `graphql-tag`.

- The `@apollo/server` package provides a full-fledged, spec-compliant GraphQL server.
- The `graphql` package provides the core logic for parsing and validating GraphQL queries.
- The `graphql-tag` package provides the gql template literal that we'll use in a moment.

```bash
npm install @apollo/server graphql graphql-tag
```

## Create Schema

```ts
// src/schema.ts

import gql from 'graphql-tag';

export const typeDefs = gql`
  type Query {
    "Get tracks array for homepage grid"
    tracksForHome: [Track!]!
  }

  "A track is a group of Modules that teaches about a specific topic"
  type Track {
    id: ID!
    "The track's title"
    title: String!
    "The track's main author"
    author: Author!
    "The track's main illustration to display in track card or track page detail"
    thumbnail: String
    "The track's approximate length to complete, in minutes"
    length: Int
    "The number of modules this track contains"
    modulesCount: Int
  }

  "Author of a complete Track"
  type Author {
    id: ID!
    "Author's first and last name"
    name: String!
    "Author's profile picture url"
    photo: String
  }
`;
```

## Initialize Apollo Server

```ts
// src/index.ts

import { ApolloServer } from '@apollo/server';
import { typeDefs } from './schema';
import { startStandaloneServer } from '@apollo/server/standalone';

async function startApolloServer() {
  const server = new ApolloServer({ typeDefs });
  const { url } = await startStandaloneServer(server);
  console.log(`
      🚀  Server is running!
      📭  Query at ${url}
    `);
}

startApolloServer();
```

## Mocking Data

```bash
npm install @graphql-tools/mock @graphql-tools/schema
```

**index.ts**

```ts
const mocks = {
  Query: () => ({
    tracksForHome: () => [...new Array(6)],
  }),
  Track: () => ({
    id: () => 'track_01',
    title: () => 'Astro Kitty, Space Explorer',
    author: () => {
      return {
        name: 'Grumpy Cat',
        photo:
          'https://res.cloudinary.com/dety84pbu/image/upload/v1606816219/kitty-veyron-sm_mctf3c.jpg',
      };
    },
    thumbnail: () =>
      'https://res.cloudinary.com/dety84pbu/image/upload/v1598465568/nebula_cat_djkt9r.jpg',
    length: () => 1210,
    modulesCount: () => 6,
  }),
};
```

**index.ts**

```diff
async function startApolloServer() {
- const server = new ApolloServer({ typeDefs });
+ const server = new ApolloServer({
+   schema: addMocksToSchema({
+     schema: makeExecutableSchema({ typeDefs }),
+     mocks,
+   }),
+ });
  const { url } = await startStandaloneServer(server);
  console.log(`
      🚀  Server is running!
      📭  Query at ${url}
    `);
}
```
