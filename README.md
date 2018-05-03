This feature has been merged in `haxe-redux`, and will be available starting next release (current version being `0.5.1`). It is already available using `haxelib git redux https://github.com/elsassph/haxe-redux`.

# Haxe Redux Thunk

Redux thunks (from [redux-thunk](https://github.com/gaearon/redux-thunk), more informations on thunks available [here](https://stackoverflow.com/questions/35411423/how-to-dispatch-a-redux-action-with-a-timeout/35415559#35415559)) are not really compatible with haxe-redux, since they work by dispatching functions instead of actions and catching them in a middleware.

This library reproduces this behavior, but with actual haxe-redux actions and a haxe-redux standard middleware.
This works by encapsulating your functions in an instance of the `redux.thunk.Thunk` enum.

There is no need for redux-thunk  runtime library, this is "native" haxe-redux code.

## Usage

### Installation

Using haxelib:
`haxelib install redux-thunk`

### Add middleware to your store

You have to add the `redux.thunk.ThunkMiddleware` for your thunks to be applied.

With haxe-redux way of using middlewares:
```haxe
var middleware = Redux.applyMiddleware(
	mapMiddleware(Thunk, new ThunkMiddleware())

	// Or, if you want to add custom data like it is possible with redux-thunk:
	// mapMiddleware(Thunk, new ThunkMiddleware({custom: "data"}))
	// You can them consume this data with Thunk.WithParams (see below)
);

createStore(rootReducer, initialState, middleware);
```

### Create your thunks

The usual way of creating thunks is creating a dedicated class with public static functions returning `redux.thunk.Thunk` enum instances:
```haxe
class TodoThunk {
	public static function add(todo:String) {
		// Encapsulate your thunk function in a `Thunk.Action` enum to create
		// an action recognized by `ThunkMiddleware`
		// Note that typing here is optional, but can help with autocompletion
		// and proper state typing
		return Thunk.Action(function(dispatch:Dispatch, getState:Void->AppState) {
			var todos = getState().todoList.todos;

			if (Lambda.exists(todos, function(t) return t.text == todo))
				return null;

			return dispatch(TodoAction.Add(todo));
		});
	}

	public static function dummy() {
		// Use `Thunk.WithParams` instead of `Thunk.Action` if you want to
		// consume the custom data from your thunk
		return Thunk.WithParams(function(dispatch, getState, params) {
			trace(params); // {custom: "data"}
			return null;
		});
	}
}
```

### Use your thunks in a container

You can use your thunks in the `mapDispatchToProps` function of your containers:
```haxe
static function mapDispatchToProps(dispatch:Dispatch) {
	return {
		addTodo: function(todo:String) return dispatch(TodoThunk.add(todo))
	};
}
```

