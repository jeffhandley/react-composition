# Usage With Redux

React-Composition allows you to very easily compose Redux-driven components into pages.  Beyond the Hello World example, only a few additions need to be made.

1. Introduce your reducers and action creators
1. Introduce a Container component that wraps Redux around your Presentational component
1. Create the Redux store in both the server and client
1. Bind your action creators for exposure on the component's external API

The example below creates a Counter page that allows the user to click a button to increment the counter.  Additionally, a querystring value is read on the server to initialize the state of the counter.

**Reducers: `reducers.js`**

``` js
export default (state = 0, action) => {
    if (action.type === 'INCREMENT') {
        return state + action.count;
    } else {
        return state;
    }
}
```

**Action Creators: `actionCreators.js`**

``` js
export default {
    increment: (count = 1) => ({
        type: 'INCREMENT',
        count: count
    })
};
```
**Presentational Component: `components/Counter.jsx`**

``` jsx
import React from 'react';

export default React.createClass({
    propTypes: {
        increment: React.PropTypes.func.isRequired,
        value: React.PropTypes.number.isRequired
    },

    handleIncrement() {
        this.props.increment();
    },

    render() {
        return (
            <div>
                Counter Value: { this.props.value }
                <div>
                    <button onClick={ this.handleIncrement }>
                        Increment
                    </button>
                </div>
            </div>
        );
    }
});
```

**Container Component: `containers/Counter.jsx`**

``` jsx
import React from 'react';
import Counter from '../components/Counter';
import actionCreators from '../actionCreators';
import { connect, Provider } from 'react-redux';

const mapStateToProps = (state) => ({
    value: state
});

const ConnectedCounter = connect(mapStateToProps, actionCreators)(Counter);

export default React.createClass({
    propTypes: {
        store: React.PropTypes.object.isRequired
    },

    render() {
        return (
            <Provider store={ this.props.store }>
                <ConnectedCounter />
            </Provider>
        );
    }
});
```

**Server: `index.js`**

``` jsx
import React from 'react';
import url from 'url';
import Counter from './containers/Counter';
import reducers from './reducers';
import actionCreators from './actionCreators';
import template from '../../templates/full-page';
import { bindActionCreators, createStore } from 'redux';
import { RenderContainer } from 'react-composition';

export default (req, callback) => {
    const { to = 0 } = url.parse(req.url, true).query;
    const store = createStore(reducers, Number(to));
    const actions = bindActionCreators(actionCreators, store.dispatch);

    template(req, (Template, templateActions) => {
        callback(
            React.createClass({
                render() {
                    return (
                        <Template
                          title='Redux Counter'
                          body={
                            <RenderContainer
                              clientSrc='/client/pages/counter-redux.js'
                              id='counter-redux'
                              state={store.getState()}>
                                <Counter store={ store } />
                            </RenderContainer>
                          }
                        />
                    );
                }
            }),
            { ...templateActions, ...actions }
        );
    });
}
```

**Client: `client.js`**

``` jsx
import React from 'react';
import ReactDOM from 'react-dom';
import Counter from './containers/Counter';
import reducers from './reducers';
import { createStore } from 'redux';
import { getRenderState } from 'react-composition/client';

const state = getRenderState('counter-redux');
const store = createStore(reducers, state);
const container = document.getElementById('counter-redux');

ReactDOM.render(<Counter store={store} />, container);
```