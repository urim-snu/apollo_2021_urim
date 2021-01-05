# Apollo_2021_urim

Movie App (Tutorial for React, GraphQL, Apollo)

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

## Local State

<pre><code>
</code></pre>
