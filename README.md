# Apollo_2021_urim

Movie App (Tutorial for React, GraphQL, Apollo)  
By Nomad Coders (https://nomadcoders.co/react-graphql-for-beginners/)

## What we are going to use

- styled components
- react-router-dom
- react apollo

#### Why Apllo?

GraphQL로 만든 API를 가장 잘 활용할 수 있는 방법.

## Router

```javascript
import React from "react";
import { HashRouter as Router, Route } from "react-router-dom";
import Home from "../routes/Home";
import Detail from "../routes/Detail";

function App() {
  return (
    <Router>
      <Route exact path="/" component={Home} />
      <Route path="/:id" component={Detail} />
    </Router>
  );
}

export default App;
```

## Apollo GraphQL

##### 기존 REST API

URL을 통해 JSON 파일을 얻음.

##### GraphQL API

Query를 통해 정보를 얻음. (POST request?)

### Apollo Client

```javascript
// in apollo.js
import ApolloClient from "apollo-boost";

const client = new ApolloClient({
  uri: uriAddress,
});

export default client;
```

이렇게 만든 client로 react를 감싸줘야 한다.
아래 index.js에서처럼 App을 감싸주면 된다.

```javascript
// in indes.js
import client from "./apollo";
import { ApolloProvider } from "@apollo/react-hooks";

ReactDOM.render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>,
  document.getElementById("root")
);
```

### Query

이전 과정에서 playground에서 했던 query 요청을 JS로 작성하기.  
query는 component 바깥에 작성하자.  
component 내에서 useQuery 함수를 통해 Query를 사용할 수 있다. (React Hooks)

```javascript
// 아래처럼 백틱으로 감싸서 쿼리를 표현한다.
const GET_MOVIES = gql`
  {
    movies {
      id
      medium_cover_image
    }
  }
`;
```

```javascript

```

- `{ A && B && ... && X }` : A, B, ... 가 `true` 이면 X를 반환.

#### link

react를 제대로 사용하기 위해, a 태그가 아니라 link를 사용한다.
React Hooks의 일종인 useParams를 통해 URL에서 id를 얻어온다.

```javascript
// in components/Movie.js
import React from "react";
import { Link } from "react-router-dom";

export default ({ id }) => (
  <div>
    <Link to={`/${id}`}>{id}</Link>
  </div>
);
```

#### Query로 data 받아서 사용하기

query에 argument가 필요할 경우, Apollo를 위한 query를 따로 작성해야 한다.  
마치 스키마를 작성하는 것처럼!

```javascript
// in routes/Detail.js
const GET_MOVIE = gql`
  query getMovie($id: Int!) {
    movie(id: $id) {
      title
      medium_cover_image
      description_intro
      language
      rating
    }
  }
`;
```

마찬가지로 useQuery hook을 이용해 data를 가져오고, useParams를 이용해 url에서 id를 얻는다.

```javascript
const { id } = useParams();
const { loading, data } = useQuery(GET_MOVIE, {
  variables: { id: +id },
});
```

### Cache

React Apollo는 cache를 갖고 있어서 불필요한 request를 줄인다.

### Data

Data를 사용히여 return 할 때 react는 아직 load되지 않은 data를 바로 요구하므로, 삼항연산자 등을 통해 load 여부를 체크해줘야 한다.

- Optional Chaining : `{data && data.movie ? data.movie.medium_cover_image : ""}` => `{data?.movie?.medium_cover_image}`

## Local State

API로 받은 Data를 Local하게 수정하는 것.  
Apollo cache 내부에서 내가 임의로 만든 fieid를 결합 -> resolvers

```javascript
// in apollo.js
const client = new ApolloClient({
  uri: uriAddress,
  resolvers: {
    Movie: {
      isLiked: () => false,
    },
  },
});
```

위 필드를 query에도 추가해줘야 한다. (@clinet)

```javascript
// my Query
const GET_MOVIES = gql`
  {
    movies {
      id
      medium_cover_image
      isLiked @client
    }
  }
`;
```

### Mutation on Clinet

resolvers 에서 mutation 을 위한 함수를 구현하고, Query를 작성한 것과 동일하게 gql을 사용해서 mutaion을 js 내부에 작성한다.

```javascript
  resolvers: {
    Movie: {
      isLiked: () => false
    },
    Mutation: {
      likeMovie: (_, { id }, { cache }) => {
        cache.writeData({
          id: `Movie:${id}`,
          data: {
            isLiked: true,
            medium_cover_image: "lalalalal"
          }
        });
      }
    }
  }
```

@client 덕분에 client에서 resolver를 찾게 된다.

```javascript
const LIKE_MOVIE = gql`
  mutation likeMovie($id: Int!) {
    likeMovie(id: $id) @client
  }
`;
```

그리고 React Hooks인 useMutation을 통해 resolver를 frontend와 엮는다.

```javascript
const [likeMovie] = useMutation(LIKE_MOVIE, {
  variables: { id: parseInt(id) },
});
```
