pre-req reading: https://kentcdodds.com/blog/fix-the-not-wrapped-in-act-warning
conditions: experienced pre-react 18
Scenario: 

Consider a react component which does two things 
- Updates some state on a button click
- Calls an API endpoint on a button click (using tan stack)

```jsx
const MyComponent = () => {
	const [flag, setFlag] = React.useState<boolean>(false);
	const { mutate } = useApiEndpoint({
		onSuccess: () => {
			updateFlag();
			console.log('Hooray!');
		},
		onError: () => {
			updateFlag();
			console.log('Oh no!');
		}
	})

	const updateFlag = () => {
		setFlag((prev) => !prev);
	}

	const handleClick = () => {
		console.log('handling click');
		mutatue();
	}

	return (
		<div>
			<Button onClick={() => handleClick()}>
				ClickMe
			</Button>
			{flag && (<div>HelloWorld</div>)}
		</div>
	)
}
```

A test would then look something like this

```jsx
test('can handle api endpoint failures', async () => {
	// mock endpoint 
	mockedApiEndpoint.mockImplementationOnce(() => {
		throw new Error('Oh No!')
	});
	// render the component
	render(<TestComponent/>);

	// click the button
	await clickButton('ClickMe');

	// expect hello world to appear
	await waitFor(() =>  
	    expect(screen.queryByText('HelloWorld')).toBeInTheDocument()  
	);

	// expect endpoint to be called
	expect(mockedApiEndpoint).toBeCalledTimes(1);
});
```

This looks fine right? But when the test runs we get this warning:
```
Warning: An update to MyComponent inside a test was not wrapped in act(...).

When testing, code that causes React state updates should be wrapped into act(...):
    
    act(() => {
      /* fire events that update state */
    });
    /* assert on the output */

... stack trace ...

      18 |     const updateFlag = () => {
    > 19 |         setFlag((prev) => !prev);
         |         ^
      20 |     };
```
Why is this warning occurring? The standard reason is **because we are not awaiting for an async process to finish**

But aren't we? 
- we await the userEvent to click the button
- we await the 'HelloWorld' to appear

**REASON**:
- The userEvent is 'tangled' with the async Api endpoint's onSuccess / onError logic.
- Click button >> trigger api call >> handling success / error
So we are awaiting the userEvent to click the button, but this doesn't await the consequences such as the api call and the following onError call.

**SOLUTIONS**:
1. Consider moving the state changes out of the api endpoints onSuccess / onError method.
```jsx
	const handleClick = () => {
		console.log('handling click');
		mutatue();
		console.log('also handling flag change');
		updateFlag();
	}
```

2. If you can't do that, can you mock the state change functions to do nothing?
```jsx
expect(mockedupdateFlag).toBeCalledTimes(1);
```

3. If you can't do that because it genuinely belongs inside the onSuccess / onError call, then wrap the userEvent in another await call.
```jsx
await waitFor(() => 
	await clickButton('ClickMe');
);
```

NOTE:
- as unhelpful as the warning is with making it obvious that the awaited userEvent is triggering another async event that is not within the bounds of the userEvent's await it does push us to take a harder look at the placement of code and reconsider the placement `updateFlag()` in the code.