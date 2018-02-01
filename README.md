# babel-plugin-react-ssr

A babel plugin to do the hidden dirty work for react-ssr. It is _strongly advised_ you use this babel plugin alongside `react-ssr` to acheive seamless server-side rendering.

This plugin will find all React components with `static fetchData` methods and do the following things to them:

- Wrap it in a HOC (higher order component)
- Add a static `_ssrWaitsFor` array
- Add a static `_ssrProps` array

Read the example below if you'd like know why these hidden properties are added.

## Installation

```sh
$ npm install babel-plugin-react-ssr --save-dev
```

## Usage

Chuck me straight in the `.babelrc` and you're *done*.

```json
{
  "plugins": ["react-ssr"]
}
```

## Example

Let's assume you have a page like this, with data calls you want to server-side render:

```jsx
import React, { Component } from 'react'
import MyComponent from './components/MyComponent'

class HomePage extends Component {
  static fetchData () {
    const myThing = new Promise((resolve, reject) => {
      fetch('/api')
        .then(res => res.json())
        .then(resolve)
        .catch(reject)
    })

    return {
      title: someApiCallThatReturnsATitle(),
      thing: myThing
    }
  }

  render () {
    return (
      <div>
        Here's the title prop: {this.props.title}
        {this.props.thing}
        <MyComponent />
      </div>
    )
  }
}

export default HomePage
```

Let's also assume your `MyComponent` imported in that example also has a `static fetchData` method.

The plugin will detect the `HomePage` has a `static fetchData` method and therefore carry out its three tasks:

- Wrap it in a HOC (that comes from `react-ssr`)
```js
export default _cohere(HomePage)
```

- Add a static `_ssrWaitsFor`, populating it with `MyComponent` after detecting it also has a `static fetchData`
```js
HomePage._ssrWaitsFor = [
  MyComponent
]
```

- Add a static `_ssrProps`, with object keys of the `fetchData` return to be injected into `this.props`
```js
HomePage._ssrProps = [
  'title',
  'thing'
]
```

`react-ssr` can then:
- Use the HOC client-side to execute `fetchData` methods.
- Read the `_ssrWaitsFor` property before a server-side render to simulatenously call all `static fetchData` methods.
- Read the `_ssrProps` property inside the HOC client-side, in order to quickly check whether those props are already defined on the component. This provides it with a quick and performant way to know whether to call `fetchData` or not.
