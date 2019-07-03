# Converting Redux and Connected Components

## Redux

It's usually easiest to start with the actions and action creators.  Here is a simple set of actions/action creators in JavaScript:

```javascript
export const HIDE_FLASH_MESSAGE = 'HIDE_FLASH_MESSAGE';
export const SHOW_FLASH_MESSAGE = 'SHOW_FLASH_MESSAGE';

export const hideFlashMessage = () => ({
  type: HIDE_FLASH_MESSAGE
});

export const showFlashMessage = (kind, message) => ({
  type: SHOW_FLASH_MESSAGE,
  kind,
  message
});
```

First convert the actions to either a string literal union type or a  `const enum`. Both compile to the same thing, but the `enum` will save you from having to hard-code the strings again in the reducer, so it is preferred:

```typescript
export const enum FlashActionType {
  hideFlashMessage = 'HIDE_FLASH_MESSAGE',
  showFlashMessage = 'SHOW_FLASH_MESSAGE'
}
```

Next define interfaces for each action creator. These correspond to the return values of the action creators:

```typescript
interface HideFlashAction {
  type: FlashActionType.hideFlashMessage;
}

interface ShowFlashAction {
  type: FlashActionType.showFlashMessage;
  kind: FlashKind;
  message: string;
}
```

Now gather up the interfaces in a union type and type your action creators:

```typescript
export type FlashAction = HideFlashAction | ShowFlashAction

export const hideFlashMessage = (): FlashAction => ({
  type: FlashActionType.hideFlashMessage
});

export const showFlashMessage = (kind: string, message: string): FlashAction => ({
  type: FlashActionType.showFlashMessage,
  kind,
  message
});
```

The final step is to import our newly-defined types into the reducer:

```typescript
import { FlashActionType, FlashAction } from '../actions/flashMessage';

interface FlashMessageState {
  kind?: string;
  message?: string;
  visible: boolean;  
}

const flashMessage = (state: FlashMessageState = { visible: false }, action: FlashAction): FlashMessageState => {
  switch (action.type) {
    case FlashActionType.showFlashMessage:
      return {
        ...state,
        action.message,
        action.kind,
        visible: true
      };
    case FlashActionType.hideFlashMessage:
      return {
        ...state,
        visible: false
      };
    default:
      return state;
  }
};

```

Because of the types we defined in our actions creator file, the compiler now has enough information to infer the type in each branch of this switch statement. It knows not to complain that we are accessing `action.message` in our first case-statement even though that value is not present on all `FlashActions`.

## React-Redux Thunks

In our last example, what if  `showFlashMessage` was not a standard action creator but instead a thunk like this:

```javascript
export const showFlashMessage = ({
  kind, message, hideAfterMS
}) => dispatch => {
  dispatch({
    type: SHOW_FLASH_MESSAGE,
    kind,
    message
  });
  if (hideAfterMS && _.isNumber(hideAfterMS)) {
    setTimeout(() => {
      dispatch(hideFlashMessage());
    }, hideAfterMS);
  }
};
```

How do we add types to this? `react-redux` exports a `ThunkAction` type that we can use. It looks like this:

```typescript
export type ThunkAction<R, S, E> = (dispatch: Dispatch<S>, getState: () => S, extraArgument: E) => R;
```

The only type parameter we will usually care about here is `S`, which should conform to the shape of our redux store. The other parameters are relevant only if we've loaded in some custom middleware to react-redux. Our typed thunk will look something like this:

```typescript
import { AppState } from '../store';

export const showFlashMessage = (kind: string, message: string, hideAfterMS?: number): ThunkAction<void, AppState, void> => dispatch => { ... }
```



## Connected Components

Typing connected components is straightforward if you remember two things:

1. typing a React component is really just writing interfaces for it's local state (if any) and its props
2. the props of a connected component are a union of the return value of `mapStateToProps`, `mapDispatchToProps`, and any props passed directly to the connected component, which are conventionally called `ownProps`.

So if our container looked like this:

```typescript
const mapStateToProps = ({ ui }) => ({
  kind: ui.flashMessage.kind,
  message: ui.flashMessage.message,
  visible: ui.flashMessage.visible,
});

const mapDispatchToProps = dispatch => ({
  onCloseClick: () => dispatch(hideFlashMessage())
});

export default connect(mapStateToProps, mapDispatchToProps)(FlashMessage);
```

We could make it typesafe like so:

```typescript
import { AppState } from 'datasetManagementUI/lib/types';
import { Dispatch } from 'redux';

export interface StateProps {
  kind: FlashKind;
  message?: string;
  visible: boolean;
}

export interface DispatchProps {
  onCloseClick: () => void;
}

const mapStateToProps = (state: AppState): StateProps => {
  const { ui } = state;
  return {
    kind: ui.flashMessage.kind,
    message: ui.flashMessage.message,
    visible: ui.flashMessage.visible,
  };
};

const mapDispatchToProps = (dispatch: Dispatch<AppState>): DispatchProps => ({
  onCloseClick: () => dispatch(hideFlashMessage())
});

export default connect(mapStateToProps, mapDispatchToProps)(FlashMessage);
```

And since the props of the connected `FlashMessage` component are just the return values of the two functions passed to `connect`, we can use TypeScript intersection types to reuse the types we've defined in our connector:

```typescript
import { StateProps, DispatchProps } from '../containers/FlashMessage';

type CombinedProps = StateProps & DispatchProps;

class FlassMessage extends React.Component<CombinedProps, {}> { ... }
```
