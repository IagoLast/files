The objective of this post is to establish clear guidelines on how applications should be structured so that they are scalable, easy to understand and easy to test.

# Global architecture schema

In essence we can summarise all frontend applications to a series of well-defined elements. The arrows indicate [coupling](https://connascence.io/) where the source of the arrow is coupled to the target element.

![](https://slabstatic.com/prod/uploads/fcn5vyb4/posts/images/dv4nVcw7bdfMx_jOGMav2fC6.svg)

## Types

They are the fundamental element of our application. Types are used to represent our business entities, function arguments, return objects, and in general any entity represented in our application. The types are only coupled to the language (typescript) and can depend on each other.

To avoid confusion, types are prefixed with the letter I, see [Naming conventions](https://aptopayments.slab.com/posts/xc590vpi#hgbtk-types-interfaces).

## Services

Services are groups of pure functions where the business logic of our application is defined. Generally these functions will be part of our hooks, but sometimes there is some shared logic throughout the application that needs to be extracted into services.

## API-Client

It is the piece in charge of communicating with the server. There is usually a 1-1 mapping between the functions offered by the client and the different endpoints offered by the backend. The client can only depend on the types and services.



In general, the client needs a state to be used (an api-token, a secret, jwt...).

This state can be handled through:

- Singleton -SessionStorage -React-Context

Each application will be able to choose the best way to have an instance of that client.

## Repositories

It is generally not a good idea for controllers to know the customer directly. For this reason, communication is carried out through repositories. Repositories expose pure functions with which to interact with the server.

In react we use [react-query](https://tanstack.com/query/v4). Which offers a series of very powerful functions such as caches, queries and mutations and is compatible with graphQL and REST.

## Controllers

They are responsible for maintaining the internal state of each page or component. They expose an api to the pages from which they can react to events and provide it with values computed from the state.

- Each controller will have a unique state using usePureState
- Each controller defines event handlers and exposes them to the view.
- The controller derives properties from the state and exposes them to the view.



```typescript
function useDemoController(props: IUseDemoControllerProps) {
  // Internal state
  const {state, dispatch} = usePureState({
    cardholders: [],
    isLoading: false,
  });
  
  // Helper functions connecting with services and repository
  function handleFetchCardholdersClick() {
    dispatch({isLoading: true});
    repository.listCardholers().then(cardholders => {
      dispatch({cardholders, isLoading: false});
    });
  }

  return {
    // Exposed event handler
    handleFetchCardholdersClick
    // Forwarded state props
    cardholders: state.cardholders,
    isLoading: state.isLoading,
    // Derived properties computed from the state
    numberOfCardholders: state.cardholders.length,
  }
}

```

## Pages / Components

Its function is to display data coming from the controller to the users and to capture events to send them to the controller. They also contain all the presentation logic and are in charge of organising the view-components.

- When a component is rendered on some navigation event is called a page.



## View components

They are stateless components. Its only function is to render properties and communicate user interactions through events.



# Recursive architecture

It is generally recommended to divide the software into small pieces, easy to understand, test, reusable...etc and the frontend is no exception.





![](https://slabstatic.com/prod/uploads/fcn5vyb4/posts/images/HR1Tkmc302u1qfrU2WBGVkob.svg)

Our page components will be at the same time made up of other smaller components that are at the same time made up of components forming a tree structure.

Sharing application state through this tree is one of the fundamental challenges that all frontend developers face.

![](https://slabstatic.com/prod/uploads/fcn5vyb4/posts/images/FnCJQQELu2QeqcVu_ZbjOSEL.svg)

One of the main rules in our architecture is to follow the [one-way-data-flow](https://tkssharma.gitbook.io/react-training/day-01/react-js-3-principles/one-way-data-flow),  keeping the state as far down the tree as possible, moving it up when strictly necessary.

Read more:

- [https://kentcdodds.com/blog/application-state-management-with-react](https://kentcdodds.com/blog/application-state-management-with-react)
- [https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster)
- [https://egghead.io/lessons/react-lifting-and-colocating-react-state](https://egghead.io/lessons/react-lifting-and-colocating-react-state)

## Folder structure

These architectural limitations are reflected in the folder structure. Making it predictable by perfectly reflecting the architecture of our component tree.

Each component will in turn have a &quot;components&quot; folder where all the components that are part of the parent component are contained. This rule is followed recursively until reaching the leaf nodes.

In this way we reduce the coupling of the application. Because we constrain the coupling of a node to its /components folder. We know that any change in a certain folder can only affect its direct relatives, but never its sibling nodes, or nodes that are part of another tree.

## Shared state

To share global state, session/localStorage, singletons, and React contexts will be used depending on the use case.

Each case must be discussed and evaluated independently.

## Shared components

In the event that a component has to be reused in several trees. It will extract to the Components folder. The components of this folder have to follow a very clear policy about their status.

- They must receive information through attributes.
- They must communicate information through events.
- If they have state, it must be an internal and hidden variable.
- They must be generic.

## Shared business logic

Shared business logic is performed by services. These services are pure functions and must be properly tested.

# State management

We use hooks to manage state. (Yes, no library required!)

The state must be kept local to the component and must be floated if absolutely necessary. Global state is held in memory, singletons, session-storage, local-storage, or context.

# Routing

We use [react-router](https://reactrouter.com/) to manage the routes.

A router is nothing more than a special component that will conditionally render a child component of the tree to the URL.

- All the routes are listed as constants in the right folder.

# Forms

For form management we use [react-hook-form](https://react-hook-form.com/).

# Patterns

## Container components

Incomplete

One of the most common scenarios in the frontend is to have a component that renders data from the server based on a query.

This component has 4 possible states:

- Idle
- Loading
- Error
- Success.

It is common for junior developers to only consider the success story. Therefore we use a higher order component (WithQuery) that allows us to handle all 4 cases automatically.

```typescript
export default WithQuery({
  // This component will be rendered when the query succeeds
  Component: MySuccessComponent,
  // The controller needs to expose a state variable
  useHooks: myControllerForTheComponent
})

```
