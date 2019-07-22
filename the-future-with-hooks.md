# React and Redux and Typescript with and without React hooks

This is a simplistic example of a redux-connected component.

In "WITHOUT hooks" our current way of doing this is outlined (using React 15)

In "WITH hooks" a possible future is outlined (using React 16)

## Setup

Let's set up our Redux types and everything...

### types.ts

```ts
// types.ts

// the type of the redux state
export interface ApplicationState {
  buttonDisabled: boolean
}
```

### actions.ts

Using suggestions from https://redux.js.org/recipes/usage-with-typescript

```ts
export const BUTTON_CLICKED = 'BUTTON_CLICKED';
export interface ButtonClickedAction {
  type: typeof BUTTON_CLICKED;
  name: string;
}
export const buttonClicked = (name: string, event: React.MouseEvent<HTMLButtonElement>) => ({
  type: BUTTON_CLICKED,
  name,
  event
})
```

### selectors.ts

```ts
export const getButtonDisabled = (state: ApplicationState): boolean => state.buttonDisabled;
```

## WITHOUT hooks (React 15, Redux 5.0.6)

Without hooks, we use redux's `connect` function along with a `mapStateToProps` and `mapDispatchToProps`; we have to define these types before doing anything.

Using help from https://medium.com/knerd/typescript-tips-series-proper-typing-of-react-redux-connected-components-eda058b6727d

```tsx
import React, { Component } from 'react';
import { connect, Dispatch } from 'react-redux';

// props passed in to the component
interface OwnProps {
  name: string;
}

// props from connect mapDispatchToProps
interface DispatchProps {
  onButtonClicked: (e: React.MouseEvent<HTMLButtonElement>) => ButtonClickedAction;
}

// props from connect mapStateToProps
interface StateProps {
  buttonDisabled: boolean
}

type Props = OwnProps & DispatchProps & StateProps;

class SomeButton extends Component<Props> {
  render() {
    const { name, onButtonClicked, buttonDisabled } = this.props;

    return (
      <button onClick={(e) => onButtonClicked(e)} disabled={buttonDisabled}>
        {name}
      </button>
    );
  }
}

const mapStateToProps = (state: ApplicationState): StateProps => ({
  buttonDisabled: getButtonDisabled(state)
});

const mapDispatchToProps = (dispatch: Dispatch<any>, { name }: OwnProps): DispatchProps => ({
  onButtonClicked: (event: React.MouseEvent<HTMLButtonElement>) => dispatch(buttonClicked(name, event))
})

export default connect<StateProps, DispatchProps, OwnProps>(
  mapStateToProps,
  mapDispatchToProps
)(SomeButton);
```

## WITH hooks (React 16+, Redux 7.1.0+)

With hooks, we don't need to use `connect` anymore and since our selectors etc. are already typed, we don't need to re-define the types at all.

```tsx
import React, { FunctionComponent } from 'react';
import { useSelector, useDispatch } from 'react-redux';

// props passed in to the component
interface Props {
  name: string;
}

const SomeButton: FunctionComponent<Props> = ({ name }) => {
  const dispatch = useDispatch();

  // since the selector is typed, we don't have to redefine it here
  const buttonDisabled = useSelector(getButtonDisabled);

  return (
    <button onClick={(e) => dispatch(buttonClicked(name, e))} disabled={buttonDisabled}>
      {name}
    </button>
  );
}

export default SomeButton;
```
