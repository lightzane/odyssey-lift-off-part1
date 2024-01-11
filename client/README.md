# Catstronauts - client

The starting point of the `client` code for Odyssey Lift-off I course.

## References

- https://www.apollographql.com/tutorials/lift-off-part1/07-the-frontend-app
- https://www.apollographql.com/tutorials/lift-off-part1/10-the-usequery-hook

## Getting Started

```bash
npm install graphql @apollo/client
```

## Apollo Client Setup

**src/index.tsx**

```ts
import React from 'react';
import ReactDOM from 'react-dom';
import GlobalStyles from './styles';
import Pages from './pages';
import { ApolloClient, ApolloProvider, InMemoryCache } from '@apollo/client';

const client = new ApolloClient({
  uri: 'http://localhost:4000',
  cache: new InMemoryCache(),
});

ReactDOM.render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <GlobalStyles />
      <Pages />
    </ApolloProvider>
  </React.StrictMode>,
  document.getElementById('root')
);
```

## Generating Types

We need the frontend to understand what type of data they involve. We could write out the TypeScript types manually but if we change our schema in the future, we have to remember to update our frontend as well - this means that our frontend's TypeScript types can easily get out of sync, if we're not careful!

Instead, we can look to the backend schema we've already written as our "single source of truth" for all of the types we could possibly query on the frontend. An easy way to do this, and to keep our frontend's type definitions consistent with the backend, is to use a GraphQL Code Generator.

```bash
npm install -D @graphql-codegen/cli @graphql-codegen/client-preset
```

**package.json**

```diff
"scripts": {
  "test": "vitest",
  "start": "vite",
  "build": "vite build",
+ "generate": "graphql-codegen"
},
```

Create `codegen.ts` in root directory

```ts
import { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
  schema: 'http://localhost:4000',
  documents: ['src/**/*.tsx'],
  generates: {
    './src/__generated__/': {
      preset: 'client',
      presetConfig: {
        gqlTagName: 'gql',
      },
    },
  },
};

export default config;
```

Run command in **CLI**: `npm run generate`

> Keep GraphQL server running in http://localhost:4000

```bash
lightzane@JPs-MacBook-Air client % npm run generate

> catstronauts-client@1.0.0 generate
> graphql-codegen

✔ Parse Configuration
⚠ Generate outputs
  ❯ Generate to ./src/__generated__/
    ✔ Load GraphQL schemas
    ✖
      Unable to find any GraphQL type definitions for the following pointers:
      - src/**/*.tsx
    ◼ Generate

```

If we run our `npm run generate` command now, we'll see an error: that's because currently our frontend code doesn't contain any GraphQL operations for the Code Generator to scan.

```diff
import { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
  schema: 'http://localhost:4000',
  documents: ['src/**/*.tsx'],
  generates: {
    './src/__generated__/': {
      preset: 'client',
      presetConfig: {
        gqlTagName: 'gql',
      },
    },
  },
+ ignoreNoDocuments: true,
};

export default config;
```

```bash
lightzane@JPs-MacBook-Air client % npm run generate

> catstronauts-client@1.0.0 generate
> graphql-codegen

✔ Parse Configuration
✔ Generate outputs

```

Even though we haven't written any GraphQL operations on the frontend yet, we can see how GraphQL Code Generator was able to pull information about the type of data our frontend will be working with.

From now on, we want to know if there aren't any GraphQL documents for the Code Generator to scan, so we can clean up our `codegen.ts` file by removing the `ignoreNoDocuments` flag. (Running `npm run generate` will still produce an error, but we'll take care of that in the next lesson by adding our first GraphQL operation!)

## A query in a component

The code for our tracks page lives in `src/pages/tracks.tsx`. At the moment, the page just displays the bare layout that we've seen previously. Let's add a query definition to it.

```tsx
import { gql } from '../__generated__/';

/** TRACKS query to retrieve all tracks */
const TRACKS = gql(`
    query GetTracks {
      tracksForHome {
        id
        title
        thumbnail
        length
        modulesCount
        author {
          id
          name
          photo
        }
      }
    }
  `);
```

Now that our frontend code contains an actual GraphQL operation, we can run our `npm run generate` function again and let the GraphQL Code Generator scan and anticipate the operations that our app will be sending. It will use this information to determine the TypeScript types for our operations

```bash
lightzane@JPs-MacBook-Air client % npm run generate

> catstronauts-client@1.0.0 generate
> graphql-codegen

✔ Parse Configuration
✔ Generate outputs
```

## Executing with `useQuery`

```ts
import { useQuery } from '@apollo/client';

const Tracks = () => {
  const { loading, error, data } = useQuery(TRACKS);

  if (loading) return 'Loading...';

  if (error) return `Error! ${error.message}`;

  return <Layout grid>{JSON.stringify(data)}</Layout>;
};
```
