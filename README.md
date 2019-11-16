[![NPM Status][npm-image]][npm-url]
[![GitHub license][license-image]][license-url]
[![LGTM Status][lgtm-image]][lgtm-url]
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/7588b3bdb457430687e3688bf2cd6121)](https://www.codacy.com/manual/subash.adhikari/nextjs-with-apollo)
[![Dependencies](https://img.shields.io/david/adikari/nextjs-with-apollo?logo=dependabot)](https://img.shields.io/david/adikari/nextjs-with-apollo)

# ⚓ nextjs-with-apollo
Apollo HOC for NextJS.


## Install

Install the package with npm

```sh
npm install nextjs-with-apollo
```

or with yarn

```sh
yarn add nextjs-with-apollo
```

## Basic Usage

1. Create a HOC 

Create the HOC using a basic setup.

```js
// hocs/withApollo.js
import withApollo from 'nextjs-with-apollo';
import ApolloClient from 'apollo-client';
import { InMemoryCache } from 'apollo-cache-inmemory';

const GRAPHQL_URL = 'https://your-graphql-url';

const createApolloClient = ({ initialState, headers }) =>
    new ApolloClient({
      uri: GRAPHQL_URL,
      cache: new InMemoryCache().restore(initialState || {}) // hydrate cache
    });
    
export default withApollo(createApolloClient);
```
Parameters `initialState` and `headers` are received in the hoc. 

If the render is happening in server, all headers received by the server can be accessed via `headers`.
If the render is happening in browser, we hydrate the client cache with the initial stated created in server.

1. Now use the HOC

```js
import React from 'react';
import { useQuery } from '@apollo/react-hooks';

import withApollo from 'hocs/withApollo';

const QUERY = gql`
  query Profile {
    profile {
      name
      displayname
    }
  }
`;

const ProfilePage = () => {
  const { loading, error, data } = useQuery(PROFILE_QUERY);

  if (loading) {
    return <p>loading..</p>;
  }

  if (error) {
    return JSON.stringify(error);
  }

  return (
    <>
      <p>user name: {data.profile.displayname}</p>
      <p>name: {data.profile.name}</p>
    </>
  );
};

export default withApollo(ProfilePage);

```

Thats all. Now Profile page will be rendered in the server. You do not need to do anything in `getInitialProps`. All queries are resolved in the sever.

If you dont want to SSR the above page then you can pass `{ssr: false}` to the hoc.

```
export default withApollo(ProfilePage, { ssr: false });
```

If you want, you can also access instance of `apolloClient` in `getInitialProps`.

```js
ProfilePage.getInitialProps = ctx => {
  const apolloClient = ctx.apolloClient;
};
```

## SSR with auth

Often graphQL server requires `AuthorizationToken` for authorizing requests. We can use the headers received in server to parse token from client side cookies.

```js
// hocs/withApollo.js
import withApollo from 'nextjs-with-apollo';
import fetch from 'isomorphic-unfetch';
import { InMemoryCache } from 'apollo-cache-inmemory';
import ApolloClient from 'apollo-client';
import { HttpLink } from 'apollo-link-http';
import { ApolloLink } from 'apollo-link';
import { setContext } from 'apollo-link-context';
import cookie from 'cookie';
import get from 'lodash/get';

const isServer = typeof window === 'undefined';

const getToken = headers => {
  const COOKIE_NAME = 'your_auth_cookie_name'
  const cookies = isServer ? get(headers, 'cookie', '') : document.cookie;

  return get(cookie.parse(cookies), COOKIE_NAME, '');
};

const attachAuth = headers => () => {
  const token = getToken(headers);

  return {
    headers: {
      authorization: `Bearer ${token}`
    }
  };
};

const createApolloClient = ({ initialState, headers = {} }) => {
  const authLink = () => setContext(attachAuth(headers));

  const httpLink = new HttpLink({
    credentials: 'include',
    uri: GRAPHQL_ENDPOINT,
    fetch
  });

  return new ApolloClient({
    ssrMode: isServer,
    link: ApolloLink.from([authLink(), httpLink]),
    cache: new InMemoryCache().restore(initialState || {})
  });
};

export default withApollo(createApolloClient);
```

## License
Feel free to use the code, it's released using the MIT license.

[npm-image]:https://img.shields.io/npm/v/nextjs-with-apollo.svg
[npm-url]:https://www.npmjs.com/package/nextjs-with-apollo
[license-image]:https://img.shields.io/github/license/adikari/nextjs-with-apollo.svg
[license-url]:https://github.com/adikari/nextjs-with-apollo/blob/master/LICENSE

[lgtm-image]:https://img.shields.io/lgtm/grade/javascript/g/adikari/nextjs-with-apollo.svg?logo=lgtm&logoWidth=18
[lgtm-url]:https://lgtm.com/projects/g/adikari/nextjs-with-apollo/context:javascript
