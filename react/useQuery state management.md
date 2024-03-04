With onSuccess and onError deprication the new normal is to directly pass the results of the query that is exposed by the useQuery hook to what ever component needs it. 

If you were triggering something like a notification that doesn't care about the data but about the actually success/failure of a request you need to setup a useEffect listening to isSuccess / isError and other exposed boolean values.

UseEffect does seem like an anti pattern but in this case we are triggering behaviour as a response to something outside the normal react cycle, (async network request).

What about doing some side effects within the query function passed to useQuery? 
its considered an anti pattern to be setting state or syncing state off react-query instead of just staying within the limits of what react-query's useQuery hook exposes. https://github.com/TanStack/query/discussions/5279#discussioncomment-5641482  

Why is onSuccess going away? https://codesandbox.io/p/devbox/sad-lichterman-cwcc44 in this example we see that todoCount (ill call this FooState sometimes) is momentarily incorrect. Why? Because the queryFunction fires, then schedules onSuccess, the useQuery hook exposes the data, and the consumer of that hook gets the fresh data immediately, while onSuccess fires, the setState is also scheduled. This results in the consuming component having fresh data, while the setState has yet to be executed. Putting any components that depend on FooState out of sync. This can lead to hard to spot bugs when testing as the data is momentarily incorrect. 
- A useEffect hook would work fine bc you can track isFetching, then add a conditional (if (!isFetching && data) { // do things...}), so when isFetching is momentarily true, useEffect see's that but the conditional doesn't pass, then when the query is complete useEffect would see isFetching change to false, this would pass the conditional and fire any logic.

What if i need to modify the data object that is exposed by useQuery? 
restructure your implementation so that the response from useQuery is used to initialise some useState for a component. Or the response from useQuery is stored in some useState via useEffect 

sources 
https://github.com/TanStack/query/discussions/5279
https://tkdodo.eu/blog/react-query-and-forms
https://tkdodo.eu/blog/breaking-react-querys-api-on-purpose 