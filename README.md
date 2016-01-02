# Elm Transit Router

Drop-in router with transitions for animated, single page apps.

    elm package install etaque/elm-transit-router


## Features

* Takes care of the *history* interaction and actions plumbing
* Enables animated transition on route changes, thanks to [elm-transit](http://package.elm-lang.org/packages/etaque/elm-transit/latest)
* Provides a simple `Signal.Address String` for navigation 


## Usage

To be able to provide animated route transitions, **TransitRouter** (and **Transit** underneath) works by action delegation: it will directly emit `Effects Action` by knowing how to wrap his own `TransitRouter.Action` type into your app's `Action` type. This is why the `actionWrapper` is needed in config below.


### Model

Say you have an `Action` type wrapping `TransitRouter.Action`:

```elm
type Action = Foo | ... | RouterAction TransitRouter.Action | NoOp
```

Also a `Route` type to describe all routes in your apps:

```elm
type Route = Home | ... | NotFound | EmptyRoute
```

You must then extend your model with `WithRoute` on `Route` type:

```elm
type alias Model = TransitRouter.WithRoute Route 
  { foo: String }
```

Your `Model` is now enabled to work with `TransitRouter`. Initialize it with an empty route that should render nothing in your view, to avoid content flashing on app init.

```elm
initialModel : Model
initialModel =
  { transitRouter = TransitRouter.empty EmptyRoute
  , foo: ""
  }
```


### Update

A config should be prepared:

```elm
routerConfig : TransitRouter.Config Route Action Model
routerConfig :
  { mountRoute : Route -> Route -> Model -> (Model, Effects Action)
  , getDurations : Route -> Route -> Model -> (Float, Float)
  , actionWrapper : TransitRouter.Action -> Action
  , routeDecoder : String -> Route
  }
```

* In `mountRoute`, you should describe what should be done in your `update` when a new route is mounted. The `Route` params are previous and new routes.

* In `getDurations`, you should describe what are the transition durations, given previous/new route and current model. Write `\_ _ _ -> (50, 200)` if you always want an exit of 50ms then an enter of 200ms. You `mountRoute` will happend at the end of exit.

* `actionWrapper` is used to transform internal `TransitAction.Action` to your own `Action`.

* `routeDecoder` takes the current path as input and should return the associated route.
See my package [elm-route-parser](http://package.elm-lang.org/packages/etaque/elm-route-parser/latest) for help on this.


It's now time to wire `init` and `update` functions:

```elm
init : String -> (Model, Effects Action)
init path =
  TransitRouter.init routerConfig path initialModel
```

This will parse and mount initial route on app init. You can get initial path value by setting up a `port` in main and provide current path from JS side.

```elm
update : Action -> Model -> (Model, Effects Action)
update action model =
  case action of

    NoOp ->
      (model, Effects.none)

    RouterAction routeAction ->
      TransitRouter.update routerConfig routeAction model

    Foo ->
      (doFoo model, Effects.none)
```


### View
