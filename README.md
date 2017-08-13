# React modern ecosystem survey 2017

    * React components
    * React flow
    * HOCs, recompose library
    * Redux flow
    * Redux parts - constants, action creators, reducers, middleware, store/state, selectors (reselect)
    * Epics - rxjs middleware
    * Structuring your project - redux ducks
    
## Few words about React
It's just a view layer for your applications. React allows you to sync up your data with DOM.
Most tools have very simple API - learn JS programming, not tools.

## React components
Stateless component - pure function. Render data from props. Use it for representation purpose.
![](./pictures/stateless-component.png)
````
import React from 'react'
const AwesomeLabel = props => <label className="my-label-style">{props.labelText}</label>
````
Stateful component - class, extends React.Component. Has its own state. Updates state itself.
![](./pictures/statefull-component.png)
```
import React, { PureComponent } from 'react'
class AwesomeInput extends PureComponent {
    state = { inputValue: '' }
    
    render() {         
            return(
                <input 
                    type="text"
                    onChange={event => this.setState({ inputValue: event.target.value })} 
                    value={this.state.inputValue}
                    />)
    }
}
```
Container component - wrapper component that passes props/state to multiple stateless components.
![](./pictures/container-component.png)
        
## React flow 
Mostly one-way data flow. Data passing by props from component to component. 
Component could have internal state and invokes render() function when state/props changed.

![](./pictures/react-flow-props.png)


![](./pictures/react-flow-state.png)

````
import React, { PureComponent } from 'react'
import PropTypes from 'prop-types'
class AwesomeInput {
    static propTypes = {
        placeholder: PropTypes.string.isRequired,
        setParentComponentValueCallback: PropTypes.func.isRequired,
        isMountedCallback: PropTypes.func.isRequired,
    }
    
    state = { inputValue: '' }
    
    handleChange = event => {
        this.setState({ inputValue: event.target.value },
            () => this.props.setParentComponentValueCallback(event.target.value)
        );
    }
        
    render() {
        this.props.isMountedCallback();
        return(
            <input 
                type="text"
                placeholder={this.props.placeholder}
                value={this.state.inputValue}
                onChange={this.handleChange}                
            />)
    }
}
````
