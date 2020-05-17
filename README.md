# ngrx-helpers

A library to simplify ngrx implementation in Angular Apps.

## Getting started

For complete reference, look at the [Example App](https://github.com/madhusudhand/ngrx-helpers/tree/master/src).

### App Module configuration

import `NgrxHelperModule.forRoot()` in app.module along with other ngrx modules.

### Effects

The helper class `NgrxEffect` simplifies the effect configuration. It lets you configure effects with minimal inputs such as action, endpint and http method.

```typescript
@Injectable()
export class UserEffects extends NgrxEffect {

  constructor(
    action$: Actions,
    httpClient: HttpClient
  ) {
    super(action$, httpClient);
  }

  @Effect() userInfo = super.effect({
    action: APP_ACTIONS.GET_USER,
    type: 'get',
    endpoint: 'https://api.github.com/users/{0}',
  });
}
```

Every effect emits three actions \(_\_RESOLVING,_ \_RESOLVED, \*\_ERROR\) as follows.

* **GET\_USER\_RESOLVING**: emitted immediately when the action GET\_USER dispatched which helps setting corresponding loaders.
* **GET\_USER\_RESOLVED**: emitted when the api gets successfully resolved.
* **GET\_USER\_ERROR**: emitted when api fails with 4XX or 5XX status codes.

**Note:** endpint can be configured with params which can be sent at the time of dispatching action.

### Reducer

```typescript
export interface UserState {
  readonly userInfo: NgrxObject<any>;
}

const defaultState: UserState = {
  userInfo: {
    state: DATA_STATE.INITIAL,
    data: null
  },
};

export function UserReducer(state = defaultState, action) {
  switch (action.type) {
    case APP_ACTIONS.GET_USER_RESOLVING:
      return StoreUtil.setResolving(state, 'userInfo', null);

    case APP_ACTIONS.GET_USER_RESOLVED:
      return StoreUtil.setResolved(state, 'userInfo', action.payload.data);

    case APP_ACTIONS.GET_USER_ERROR:
      return StoreUtil.setError(state, 'userInfo', 'Error Message');

    default:
      return state;
  }
}
```

### Dispatch action

```typescript
this.store.dispatch({
  type: APP_ACTIONS.GET_USER,
  payload: {
    pathParams: ['octocat'],
    // queryParams: [{ name: 'value' }],
  },
});
```

### Subscribe a state

Subscriptions are simplified by extending the component from the helper class `NgrxStoreSubscription`.

```typescript
export class AppComponent extends NgrxStoreSubscription implements OnInit {
  userInfo = {};

  constructor(private store: Store<any>) {
    super(store);
  }

  ngOnInit() {
    super.getState({
      feature: 'APP', // the feature of the reducer
      reducer: 'USER_REDUCER', // name of the reducer
      state: 'userInfo', // name state from the store
    }).subscribe((data) => {
      this.userInfo = data;
    });

    // refer to the example app to know how reducers registered for a feature
  }
}
```

### Managing Template

Every reducer gets resolved with data with the following format.

```typescript
{
  state: DATA_STATE, // which can be RESOLVING, RESOLVED, ERROR
  data: any, // data which is set in the reducer
}
```

This format helps us set the appropriate view in the component template.

```markup
<div [ngrxView]="userInfo.state">
  <div *ngrxViewResolving>Loading...</div>
  <div *ngrxViewError>Error fetching user info</div>
  <div *ngrxViewResolved>
    {{userInfo.data | json}}
  </div>
</div>
```

with the help of directives, component automatically responds based on the state from the store.

### Summary

When the action _GET\_USER_ gets dispatched

* it emits **GET\_USER\_RESOLVING**
  * which will set the `state` in store to 'RESOLVING'
  * the component will render **\*ngrxViewResolving**. \(a loader screen\)
* if api call fails, it emits **GET\_USER\_ERROR**
  * which will set the `state` in store to 'RESOLVED'
  * the component will render **\*ngrxViewError**. \(error screen\)
* if api call fails, it emits **GET\_USER\_RESOLVED**
  * which will set the `state` in store to 'RESOLVED'
  * the component will render **\*ngrxViewResolved**. \(appropriate UI for user data\)

