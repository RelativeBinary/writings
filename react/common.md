Avoid generic imports 
```
// do
import Thing from '@dep/allThings/Thing'

// dont 
import { Thing } from '@dep/allThings'
```
has performance issues if done incorrectly
https://mui.com/material-ui/guides/minimizing-bundle-size/ 
https://github.com/eslint/eslint/discussions/17047
Typically resolved with this 
```
  "rules": {
    "unused-imports/no-unused-imports": "error",
    "no-restricted-imports": [
      "error",
      {
        "name": "@mui/material",
        "message": "Please use \"import foo from '@mui/material/foo'\" instead."
      }, {
        "name": "@mui/icons-material",
        "message": "Please use \"import foo from '@mui/icons-material/foo'\" instead."
      }
    ]
  },
```

Hooks and functions
- can we make these hard coded values something that passed in as a param or a value / func that is declared outside the component?

Components
- extracting logic into regular ts/js functions can make testing them easier and remove the need to test with the component.

Typescript type identification
- INFO: can id the right type for whats expected by reading the error message and clicking through to what the library wants

Props and typescript interfaces 
- when using multiple interfaces we can source the values of the extended interfaces through `props` rather than creating new props to access those values 
- There are times where you may need to create a whole new prop to be passed to a inner component despite extending that inner components interface. In these cases its fine, because they serve a purpose unrelated to the former scenario.
```
// Do
interface A extends B, C {
	propAFoo: string;
}

const MyComponent = ({propAFoo, ...props}:A ) => {
	return (
		<AThing propAThing={propAFoo}>
			<BThing propBThing={props.BThingOnChange} />
		</AThing>
	)
}

// Dont
interface A extends B, C {
	propAFoo: string;
	onChange: () => void
}

const MyComponent = ({propAFoo, onChange}: A) => {
	return (
		<AThing propAThing={propAFoo}>
			<BThing propBThing={onChange}
		</AThing>
	)
}

```

Testing
- don't await assert something that hasn't changed the test will assume the state updates have finished if the condition is immediately met. causing checks for other stuff that is ACTUALLY CHANGING to fail.
```
await waitFor(() => expect(screen.getByText(/exceptions: 2/i)).toBeInTheDocument());

await waitFor(() => expect(mockedExceptionSearch).toHaveBeenCalledTimes(1));  
await waitFor(() => expect(screen.getByText(/1–2 of 2/i)).toBeVisible());

// do some user action

await waitFor(() => expect(screen.getByText(/exceptions: 2/i)).toBeInTheDocument());

await waitFor(() => expect(mockedExceptionSearch).toHaveBeenCalledTimes(2));  
await waitFor(() => expect(screen.getByText(/3-4 of 20/i)).toBeVisible());
```
