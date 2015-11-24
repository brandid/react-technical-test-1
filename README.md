### React Technical Test for Try.com Frontend Engineers

This question 
* is for candidates already familiar with React
* should take you no longer than 2 hours and shorter if you are already familiar with `react-motion`

You are highly encouraged to make the code below run. Feel free to reach out if you have questions.

###### Problem

Your designer thinks he can write javascript so he put together the following code which has a performance issue.
The code uses `react-motion` to animate blocks in and out of the page. It's a good effort, but the animation is slow and jittery. Your job is to make it buttery smooth.

###### Safe to Assume
* On page load, the Demo component correctly renders 3 blocks with their corresponding title text.
* When you click a block, it animates out of the DOM
* But the animation is too jittery and way too slow
* The `ExpensiveToRenderBlockComponent` is already fully optimized
* There is no problem with way `react`, `react-motion` or `TransitionMotion` work, the problem is in our code below

###### Hints
* If you are not familiar with `react-motion`, the TransitionMotion component expects a function as its child. That function is called with argument `interpolatedStyles` hundreds of times a second. In this case, `react-motion` interpolates the opacity between 0 and 1 using a spring algorithm (hundreds of times a second). On every call, we take the interpolated opacity value and apply it to the `AnimatingContainer`, so the `ExpensiveToRenderBlockComponent`'s container has an animating opacity. For the user, `ExpensiveToRenderBlockComponent` doesn't visually change from render to render (except that its parent container has animating opacity, of course).
* The problem is in the Demo render() function

###### Questions
* Why is the animation jittery?
* How would you fix this?

```
import React from 'react';
import { TransitionMotion } from 'react-motion';
import ExpensiveToRenderBlockComponent from '...someInternalComponent';

const Demo = React.createClass({
  getInitialState() {
    return {
      blocks: {
        a: {
          'Block Title A'
        },
        b: {
          'Block Title B'
        },
        c: {
          'Block Title C'
        },
      },
    };
  },

  getStyles() {
    let configs = {};
    Object.keys(this.state.blocks).forEach(key => {
      configs[key] = {
        opacity: spring(1),
        title: this.state.blocks[key], // not interpolated
      };
    });
    return configs;
  },

  // not used here. We don't add any new items
  willEnter(key) {
    return {
      opacity: spring(0), // start at 0, gradually expand
      title: this.state.blocks[key], // this is really just carried around so
      // that interpolated values can still access the title when the key is gone
      // from actual `styles`
    };
  },

  willLeave(key, previousStyle) {
    return {
      opacity: spring(0), // make opacity reach 0
      title: previousStyle.title,
    };
  },

  handleClick(key) {
    const {...newBlocks} = this.state.blocks;
    delete newBlocks[key];
    this.setState({blocks: newBlocks});
  },

  render() {
    return (
      <TransitionMotion
        styles={this.getStyles()}
        willEnter={this.willEnter}
        willLeave={this.willLeave}>
        {interpolatedStyles =>
            <div>
              {Object.keys(interpolatedStyles).map(key => {
                
                const {title, ...style} = interpolatedData[key];
                const someBlock = <ExpensiveToRenderBlockComponent title={title}/>

                return (
                  <AnimatingContainer onClick={this.handleClick.bind(null, key)} style={style}>
                    {someBlock}
                  </AnimatingContainer>
                );
              })}
            </div>
          );
        }
          

      </TransitionMotion>
    );
  },
});

const AnimatingContainer extends React.Component {
  render() {
    const {
      interpolatedStyle,
      ...other
    } = this.props;

    const opacity = interpolatedStyle.opacity

    return (
      <div style={opacity: opacity} {...other}>
        {this.props.children}
      </div>
    );
  }
}
```
