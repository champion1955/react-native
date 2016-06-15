# Getting Started

[Follow the first page of the getting started guide on React Native](https://facebook.github.io/react-native/docs/getting-started.html)

**If you are on a windows machine you can only develop an Android app.**

**If you are on a Mac I recommend downloading [Nuclide](http://nuclide.io/docs/quick-start/getting-started/), an IDE created by Facebook for React Native**.


## Initializing the project

```
react-native init NativeReddit
```

## Installing the dependencies

Enter the folder you just created above, `NativeReddit`:

```
cd NativeReddit
```

Then install the following 4 modules (`--save` saves references to each module in your `package.json` file).

```
npm install redux react-redux redux-thunk isomorphic-fetch --save
```

# React Native

React Native was born out of two realities for Facebook:

1. Native Apps are: 
    1. fast
    2. they're slower to create
    3. require different code bases for each platform.
2. Hybrid apps (Webview) are fast to develop, but they aren't as performant.

With React Native you build your application in JavaScript. The Native app runs a JavaScript engine in another thread. The JavaScript-based React app communicates with the Native part of the application asychronously. In otherwords, none of the JavaScript blocks the main UI thread.

One of the motos for React Native is that it is "learn once, write anywhere" (not "write once, run anywhere". While the business logic can be shared between your React Native Android app and your iOS app the UI code will differ - sometimes not by much though. However, you can reuse your skills from one platform to the other.

# Creating the Native Reddit App

Open the `NativeReddit` folder in the IDE/text editor of your choice (e.g. Nuclide/Atom/Sublime).

`NativeReddit/index.ios.js` is the entry point to your app if you build an iOS app while `NativeReddit/index.android.js` is the entry point to your app if you build an Android app. I'll refer to the index file as `index.*.js` and if you are building an Android app you'll refer to the `index.android.js` file while if you are building an iOS app you will refer to `index.ios.js`.

## Creating our first native component

We'll start by creating a top navigation bar that will initially hold the current view's title.

In `NativeReddit` create a folder called `components` then create the file `components/reddit-nav-bar.js` (React Native does not recognize `.jsx` extensions, so we'll stick to `.js`).

First we'll import all the modules needed to build our own module:

```javascript
import React, { Component } from 'react';
import {
  StyleSheet,
  Text,
  View
} from 'react-native';
```

To create the component we'll use an `ES6` class and extend React Native's `Component` class. The class will include two methods `constructor()` and `render()`:

```javascript
class RedditNavigationBar extends Component {
  constructor(props){
    super(props);
    this.title = props.title;
  }

  render(){
    return (
      <View>
        <View style={styles.toolbar}>
          <Text style={styles.title}>{this.title}</Text>
        </View>
      </View>
    );
  }
}
```

In the constructor we pass the `props` to the parent's class using `super(props)`.

The `render()` function is what returns the UI for this component. Note that we are no longer using `<div>`'s, `<View>`'s and `<Text>` are actual native UI elements that React Native will create for us. 

Next we create the constant to hold the styles that our component requires:

```javascript
const styles = StyleSheet.create({
  toolbar: {
    backgroundColor: '#CFE3FA',
    paddingTop: 30,
    paddingBottom: 10,
    flexDirection: 'row'
  },
  title: {
    fontSize: 18,
    color: '#fff',
    textAlign: 'center',
    flex: 1
  }
});
```

React Native uses properties that are very similar to CSS and converts them to their analogues in Native. React Native also uses the Flex Box model for layout. 

`flex: 1` means that the title element should take up the width of our toolbar. If there were two elements in the tool bar and each had `flex: 1` that would mean that each gets 50% of the toolbar.

Finally, on the last line of `reddit-nav-bar.js` export our component:

```javascript
export default RedditNavigationBar;
```

## Using our new component

Open `index.*.js` and add the following import after the `'react-native'` imports:

```javascript
import RedditNavigationBar from './components/reddit-nav-bar.js';
```

Now update the autogenerated `NativeReddit` component to look like:

```javascript
class NativeReddit extends Component {
  render() {
    return (
      <View style={styles.outerContainer}>
        <RedditNavigationBar title="Reddit" />
      </View>
    );
  }
}
```

Above we use our new `RedditNavigationBar` component and pass it a title property of `"Reddit"`.

Finally, update the styles constant in `index.*.js` to look like:

```javascript
const styles = StyleSheet.create({
  outerContainer: {
    flex: 1
  },
  container: {
    flex: 1
  }
});
```

If everything is working you should see a light blue navigation bar with the title "Reddit".

## Creating our list of Reddit top stories

In `components/` create the file `components/reddit-list.js`  There are several new concepts that we'll need in order to build our list of Reddit stories. We'll improve the component incrementally.

First we'll start by importing the necessary modules and creating a constant to hold a couple Reddit stories for testing.

```javascript
import React, { Component } from 'react';
import {
  StyleSheet,
  Text,
  View,
  ListView,
  Image
} from 'react-native';

const REDDIT_STORIES = [
  {
    title: 'Got another dog yesterday (right), he\'s settling in well!',
    thumbnail: 'https://b.thumbs.redditmedia.com/5SOQ6sNiqEyqh48M9MOgM6sZmj-5iejiddLuRhPkWxA.jpg'},
  {
    title: 'We found this gal constantly stealing from our peach trees, so we named her Peaches. (Very original, I know)',
    thumbnail: 'https://b.thumbs.redditmedia.com/8ZkujfQt-MmvxoB_oQdq5DOgT3L3VkaeBDz_O40MzTk.jpg'}
];
```

Next we'll create, `RedditList`, the component that will contain a `ListView` of all our Reddit top stories:

```javascript
class RedditList extends Component {
  constructor(props){
    super(props)
    var ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2});
    this.state = {
      dataSource: ds.cloneWithRows(REDDIT_STORIES)
    };
  }

  renderRow(rowData){
    return (
      <View style={styles.row}>
        <Image style={styles.image} source={{uri: rowData.thumbnail}} />
        <View style={styles.rightColumn}>
          <Text style={styles.title}>{rowData.title}</Text>
        </View>
      </View>
    );
  }

  render() {
    return (
      <ListView
        enableEmptySections={true}
        dataSource={this.state.dataSource}
        renderRow={this.renderRow}
      />
    );
  }
}
```

In the constructor we instantiate a `DataSource` object and pass a function through that specifiies the condition for a row value changing (the condition necessary for re-rendering a row). Then we'll use the `cloneWithRows()` function, passing in our Reddit stories, to seed our dataSource. Finally, we assign this data to the component's `state` object. Components are generally more performant, and easier easier to reason, when they don't have an internal state, but sometimes it's impossible to avoid not having a state.

Finally, in the bottom of the same file we'll create the `styles` and export our module:

```javascript
const styles = StyleSheet.create({
  row: {
    marginLeft: 4,
    marginRight: 4,
    marginTop: 4,
    padding: 4,
    flex: 1,
    flexDirection: 'row',
    backgroundColor: '#fff',
    borderRadius: 2
  },
  rightColumn: {
    flex: 1,
    paddingLeft: 5
  },
  title: {
    fontSize: 16
  },
  image: {
    width: 100,
    height: 100
  }
});

export default RedditList;
```

In `index.*.js` you can import the new module:

```javascript
import RedditList from './components/reddit-list.js';
```

Now update the `render()` function in `NativeReddit` to include our `RedditList` component:

```javascript
render() {
  return (
    <View style={styles.outerContainer}>
      <RedditNavigationBar title="Reddit" />
      <View style={styles.container}>
        <RedditList />
      </View>
    </View>
  );
} 
```

If everything is working you should see two Reddit stories, each with their own thumbnail.

## Moving our nav bar to the `RedditList` Component

In our final app their will be a back button in the navigation bar. However, the back button will be conditional on what page is currently visible. There are several different ways we can accomplish this, but for the purpose of this workshop we're going to move `<RedditNavigationBar title="Reddit" />` from index.*.js into `RedditList`.

In `reddit-list.js` add the following import:

```javascript
import RedditNavigationBar from './reddit-nav-bar.js';
```

Then update the render method to:

```javascript
render() {
  return (
    <View style={styles.outerContainer}>
      <RedditNavigationBar title="Reddit" showBackButton={false}/>
      <View style={styles.container}>
        <ListView
          enableEmptySections={true}
          dataSource={this.state.dataSource}
          renderRow={(rowData) => this.renderRow(rowData)}
        />
      </View>
    </View>
  );
}
```

Update the style object to include:

```javascript
outerContainer: {
  flex: 1,
  backgroundColor: '#ccc'
},
container: {
  flex: 1,
},
```
 
 In `index.*.js` update the `render()` method to:
 
 ```javascript
 render(){
   return (
     <RedditList />
   );
 }
 ```
 
Finally remove the styles constant and Style import, as they're no longer needed in `index.*.js`.

You should still see the navbar with the list of 2 Reddit stories.
  
## Comments View

When a user clicks on a single story they should be taken to the comments view for that story. To make this work we'll first create a `RedditComments` component and then we'll integrate navigation into our app. In `components/` create `reddit-comments.js` and add the following imports and const:

```javascript
import React, { Component } from 'react';
import {
  StyleSheet,
  Text,
  View,
  ListView,
  Image
} from 'react-native';

import RedditNavigationBar from './reddit-nav-bar.js';

const REDDIT_COMMENTS = [
  {body: 'lol'},
  {body: 'ftw'}
];
```

Our `RedditComments` component will be very similar to the `RedditList` component. **Try to do this on your own - it's almost the same!**

### `RedditComment` Cheat

```javascript
class RedditComments extends Component {
  constructor(props){
    super(props);
    var ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2});
    this.state = {
      dataSource: ds.cloneWithRows(REDDIT_COMMENTS)
    };
  }

  renderRow(rowData){
    return (
      <View style={styles.row}>
        <Text style={styles.text}>{rowData.body}</Text>
      </View>
    );
  }

  render(){
    return (
      <View style={styles.outerContainer}>
        <RedditNavigationBar title="Reddit"/>
        <View style={styles.container}>
          <Text>Article!</Text>
          <ListView
            enableEmptySections={true}
            dataSource={this.state.dataSource}
            renderRow={this.renderRow}
          />
        </View>
      </View>
    );
  }
}
```

The styles and the export:

```javascript
const styles = StyleSheet.create({
  outerContainer: {
    flex: 1,
    backgroundColor: '#ccc'
  },
  container: {
    flex: 1,
  },
  row: {
    marginLeft: 4,
    marginRight: 4,
    marginTop: 4,
    padding: 4,
    flex: 1,
    flexDirection: 'row',
    backgroundColor: '#fff',
    borderRadius: 2
  },
  text: {
    flex: 1
  }
});

export default RedditComments;
```

You'll notice that we have the exact same `outerContainer` and `container` style defined in `reddit-list.js` and `reddit-comments.js`, we should move these to their own module and shared between pages. For this workshop we'll skip this step.

In `index.*.js` add the `RedditComments` import:

```javascript
import RedditComments from './components/reddit-comments.js';
```

At this point you have no way of viewing your new comments page. Let's fix that!

## App Navigation

**WARNING**: React Native's Navigation is going through some big changes, what you see below may not work in versions > 0.26.x.

In `index.*.js` update your `'react-native'` import to look like:

```javascript
import {
  AppRegistry,
  Navigator
} from 'react-native';
```

In `NativeReddit` update the render method to use the new `Navigator` element:

```javascript
render() {
  return (
    <Navigator
     initialRoute={{name: 'RedditList'}}
     renderScene={this.renderScene}
    />
  );
}
```

You'll note that we're specifying the `initialRoute` (e.g. the starting point). The name `RedditList` will make sense shortly. Next we specify a function, `this.renderScene` that will render the scene when the scene changes. We'll need to go ahead and add `renderScene` to our `NativeReddit` class.

```javascript
  renderScene(route, navigator){
      switch (route.name) {
        case 'RedditList':
          return <RedditList navigator={navigator} {...route.passProps} />
        case 'RedditComments':
          return <RedditComments navigator={navigator} {...route.passProps} />
        default:
          return <RedditList navigator={navigator} {...route.passProps} />
      }
  }
```

`renderScene` accepts to parameters: 1) `route`, which contains the routes name, and `navigator`, an object that helps us navigate through our app. The `renderScene` function is effectly a router that matches a given "scene" for a given `route.name`. It passes the `navigator` object to that seen and it passes any parameters that may have been passed from one scene to another using the spread operator `{...route.passedParams}`.

Now we'll make the rows in our `RedditList` clickable. Open up `reddit-list.js` and update the `'react-native'` import to:

```javascript
import {
  StyleSheet,
  Text,
  View,
  ListView,
  Image,
  TouchableHighlight
} from 'react-native';
```

We're adding `TouchableHeightlight` which we'll use to make our rows clickable. Now update the `renderRow` method in `RedditList` to:

```javascript
  renderRow(rowData){
    return (
      <TouchableHighlight
        onPress={() => {
          this.props.navigator.push({
            name: 'RedditComments',
            passProps: rowData
          });
        }}
        underlayColor='#ddd'
      >
        <View style={styles.row}>
          <Image style={styles.image} source={{uri: rowData.thumbnail}} />
          <View style={styles.rightColumn}>
            <Text style={styles.title}>{rowData.title}</Text>
          </View>
        </View>
      </TouchableHighlight>
    );
  }
```

Above we've added the `TouchableHighlight` element around each row. When the user presses on the row, `onPress` is called, which calls our function that pushes a new view onto the stack. The view we're pushing onto the navigation stack is the `RedditComments` view and we're passing the row's data. 

At this point you should be able to click on a story and go to it's comment view (`RedditComments`). Now we need a way to go back to the `RedditList` view.

### Updating `reddit-nav-bar.js` to support going back

Open `reddit-nav-bar.js` and update the navbar `View` to include a back button - use the `TouchableHighlight` component. **IMPORTANT**: Be user to include the the `TouchableHighlight` import in the `'react-native'` imports!!!

#### Cheat

```javascript
  render(){
    return (
      <View>
        <View style={styles.toolbar}>
          {/* Left Button */}
          <TouchableHighlight
            style={[styles.button, !this.props.back && styles.hide]}
            onPress={this.props.back}
            underlayColor='#CFE3FA'
          >
            <Text style={ styles.buttonText }>{'\u2039'} Back</Text>
          </TouchableHighlight>
          <Text style={styles.title}>{this.title}</Text>
          {/* Right Button, helps title stay centered! */}
          <TouchableHighlight style={[styles.button, styles.hide]}>
            <Text></Text>
          </TouchableHighlight>
        </View>
      </View>
    )
  }
```
We include two `TouchableHighlight` buttons because it makes the top layout much simpler and the title stays centered. It's also possible that we might want to put content in that right button. For both buttons we use the array notation to pass in multiple style objects. The second style object, `styles.hide`, is only there conditionally if the function `this.props.back` is defined. (e.g. if a function is based into `back={}` prop). This trick allows us to conditionally show/hide the back button depending on whether a function to go back is passed into `RedditNavigationBar`.

Add the following to the styles object:

```javascript
,
  button: {
    width: 80,
    backgroundColor: '#CFE3FA',
    alignItems: 'center'
  },
  buttonText: {
    fontSize:18,
    color: '#fff',
    backgroundColor: '#CFE3FA',
    padding: 0,
    margin: 0
  },
  hide: {
    opacity: 0
  }
```

At the moment our back button should be invisible. To show the back button when `RedditComments` is active open `reddit-comments.js` and update the `RedditNavigationBar`, in the `render` method, to:

```javascript
<RedditNavigationBar
  title="Reddit"
  back={this.props.navigator.pop}
/>
```

Above we're passing in the function `navigator.pop`, which is part of the `this.props` object. 

Your app's navigation should now be working!!!!

## Adding Redux

In `index.*.js` import the following:

```javascript
import { createStore, applyMiddleware, combineReducers } from 'redux';
import { connect, Provider } from 'react-redux';
import thunkMiddleware from 'redux-thunk';
```

From the previous workshop you should be somewhat familiar with `'redux'` and `'react-redux'`. We've now added `'redux-thunk'`, which allows our application to dispatch async functions, instead of just plain objects. This will be helpful for making AJAX calls to Reddit's API.

In `index.*.js`, right above your `NativeReddit` component add:

```javascript
const rootReducer = combineReducers({
  reddit: () => { [] }
});

const store = createStore(
  rootReducer,
  {},
  applyMiddleware(
    thunkMiddleware
  )
);
```

The `reddit` is just a placeholder function that returns an empty array, we'll fix that later. Finally, update the `render()` method in `NativeReddit` to:

```javascript
  render() {
    return (
      <Provider store={store}>
        <Navigator
          initialRoute={{name: 'RedditList'}}
          renderScene={this.renderScene}
        />
      </Provider>
    );
  }
```
The `Navigator` component is now surrounded by `'react-redux'`'s `Provider` component.

### Creating the redux reducers & actions

Next we're going to create the actions that will help us fetch posts and receive posts. We'll be using `'isomorphic-fetch'` which allows us to make API calls regardless of the platform we're on (web, native, server).

Create a new folder called `actions/` and create a file in it called `reddit-actions.js`.

In `actions/reddit-actions.js` place the following code:

```javascript
import fetch from 'isomorphic-fetch';

export const RECEIVE_POSTS = 'RECEIVE_POSTS';

const receivePosts = (json) => {
  return {
    type: RECEIVE_POSTS,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

const fetchPosts = () => {
  return dispatch => {
    return fetch('https://www.reddit.com/top.json')
      .then(response => response.json())
      .then(json => dispatch(receivePosts(json)))
  }
}

export { fetchPosts };
```

`receivePosts` is a relatively strait forward function, it takes a JSON payload, fetched from Reddit, and returns an action that we can dispatch to redux.

`fetchPosts` is a little more fancy! It returns an anonymous function that takes one parameter, `dispatch`. The anonymous function returns a promise. The promise is returned from fetching the API data, turning it into json, then dispatching the `receivePosts` action when it's complete. 

Create a new folder called `reducers/` then create a file in it called `reddit-reducer.js`. In `reducers/reddit-reducer.js` place:

```javascript
import { RECEIVE_POSTS } from '../actions/reddit-actions.js';

const redditStoriesReducer = (state=[], action) => {
  switch(action.type){
    case RECEIVE_POSTS:
      return action.posts;
    default:
      return state;
  }
}

export { redditStoriesReducer };
```

### Connecting redux to our `RedditList`

In `reddit-list.js` add the `react-redux` import:

```javascript
import { connect } from 'react-redux';
```

Then below the `RedditList` class, and above `const styles =` add:

```javascript
const mapStateToProps = (state) => {
  return { posts: state.reddit };
}

RedditList = connect(mapStateToProps)(RedditList);
```

This connects redux to our component. However, our component now needs to be able to handle changes to it's state (e.g. after the API call completes). We'll update the `constructor` function in `RedditList` to use `props.posts` instead of `REDDIT_STORIES`. Next we'll add a new function to `RedditList` that will handle updates to props passed into our component:

```javascript
  componentWillReceiveProps = (nextProps) => {
    this.setState({
      dataSource: this.state.dataSource.cloneWithRows(nextProps.posts)
    });
  }
```

`componentWillReceiveProps` is a "life-cycle" function that React calls when the component receives new props.


### Updating `index.*.js`

Import the reducer and actions:

```javascript
import { redditStoriesReducer } from './reducers/reddit-reducer.js';
import { fetchPosts } from './actions/reddit-actions.js';
```

Update the `rootReducer` to use the `redditReducer`:

```javascript
const rootReducer = combineReducers({
  reddit: redditStoriesReducer
});
```

Finally, at the very end of `index.*.js`, after the `AppRegistry` line add:

```javascript
// bootstrap app
(function initialize() {
  fetchPosts()(store.dispatch);
})();
```

That code will load the intial data from Reddit.

## Loading Comments

### Adding more actions

Update `reddit-actions.js` and add the following:

```javascript
export const RECEIVE_COMMENTS = 'RECEIVE_COMMENTS';

const receiveComments = (json) => {
  return {
    type: RECEIVE_COMMENTS,
    comments: json[1].data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

const fetchComments = ({subreddit, id, slug}) => {
  var path = `/r/${subreddit}/comments/${id}/${slug}.json`;
  var url = `http://www.reddit.com${path}`;

  return dispatch => {
    return fetch(url)
      .then(response => response.json())
      .then(json => dispatch(receiveComments(json)));
  }
}
```

then be sure to add `fetchComments` to the export:

```javascript
export { fetchPosts, fetchComments };
```

### Adding reducers

In `reddit-reducers.js` update the imports to:

```javascript
import { RECEIVE_POSTS, RECEIVE_COMMENTS } from '../actions/reddit-actions.js';
```

Then add the following reducer:

```javascript
const redditCommentsReducer = (state=[], action) => {
  switch (action.type) {
    case RECEIVE_COMMENTS:
      return action.comments
    default:
      return state;
  }
}
```

Finally add the new reducer to the exports:

```javascript
export { redditStoriesReducer, redditCommentsReducer };
```

In `index.*.js` update the import state for the reducers to look like:

```javascript
import { redditStoriesReducer, redditCommentsReducer } from './reducers/reddit-reducer.js';
```

Then update the `rootReducer`:

```javascript
const rootReducer = combineReducers({
  reddit: redditStoriesReducer,
  comments: redditCommentsReducer
});
```

### Connecting our `RedditComments` to redux

First import `'react-redux'` and the new action we created above:

```javascript
import { connect } from 'react-redux';
import { fetchComments } from '../actions/reddit-actions.js';
```

Below `RedditComments` add the following code to connect `RedditComments` to redux:

```javascript
const mapStateToProps = (state) => {
  return { comments: state.comments };
}

const mapDispatchToProps = (dispatch) => {
  return {
    fetchComments: (redditStory) => {
      return fetchComments(redditStory)(dispatch);
    }
  }
}

RedditComments = connect(
  mapStateToProps,
  mapDispatchToProps
)(RedditComments);
```

`mapStateToProps` maps redux's state object to the params passed into `RedditComments`. `mapDispatchToPros` allows us to pass a function into `RedditComments`, a function that has access to the `redux`'s dispatch function. 

Finally we'll update `RedditComments`. First alter dataSource line in the `constructor` to:

```javascript
dataSource: ds.cloneWithRows(this.props.comments)
```

Then add the following two functions to `RedditComments`:

```javascript
  componentDidMount() {
    this.props.fetchComments(this.props);
  };

  componentWillReceiveProps = (nextProps) => {
    this.setState({
      dataSource: this.state.dataSource.cloneWithRows(nextProps.comments)
    });
  }
```

You should be familiar with `componentWillReceiveProps` from above. `componentDidMount` is a nother life cycle function that's called after the component is mounted - it's the recommend spot for triggering AJAX calls.

