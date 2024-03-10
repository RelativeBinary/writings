# a RHF with Yup validation nullable number input that accepts integers and discrete numbers, not 'e, +, or -'
This is what I ended up with 
```
<TextDisplayAndInputField  
    type={'number'}  
    id={`${initialValue.id}-smpiGrowthPercent`}  
    editing={isEditing}  
    displayValue={getValues('smpiGrowthPercent') === 0 ? '-' : getValues('smpiGrowthPercent')}  
    endAdornment={<Percent />}  
    {...register('smpiGrowthPercent')}  
    onKeyDown={(evt) =>  
        (evt.key === 'e' || evt.key === '+' || evt.key === '-') && evt.preventDefault()  
    }  
    onPaste={(evt) => {  
        evt.preventDefault();  
        return false;  
    }}  
    helperText={formState.errors.smpiGrowthPercent?.message ?? ''}  
    error={Boolean(formState.errors.smpiGrowthPercent)}  
/>
```
This is a bit more developed but underneath its basically a text field that only shows when `editing` is true otherwise show `displayValue`
- `displayValue`, this is for the plain text display of the form value when not editing. The input component is controlled by `{...register('smpiGrowthPercent')}` for react-hook-form 
- literally every other prop is for the underlying textfield input.


Why not let yup do all the validation with `.number()` and `.nullable()` ? Lets look at an example
```
smpiGrowthPercent: yup  
    .number()  
    .transform((val) => { 
    // allows us to see what yup sees 
        console.log('val', val);  
        return val;  
    })  
    .nullable()  
    .typeError('not valid'),
    
...in the component where RHF is...
console.log('SMPI Growth Percent', getValues('smpiGrowthPercent'));
```

1. when the text field is cleared yup gets `NaN` and records the value as an empty string (val is what `transform()` see's `SMPI Growth Percent` is what RHF gets)
![[Pasted image 20240310211603.png]]
![[Pasted image 20240310212105.png]]

2. an invalid number also gives yup `NaN` and RHF gets an empty string (note: once invalid more inputs will not re-trigger validation as console.log fires only once after `NaN`)
![[Pasted image 20240310212316.png]]
![[Pasted image 20240310212240.png]]

This means that we cannot tell if what the user entered is something nonsensical like `-+1e`  or just wanted the value to be `0`

This leaves a few more options
- Don't let the user enter `e, -, or +` BUT the user can still *paste* invalid characters
	- then block paste, starting to feel hacky...
```
    onKeyDown={(evt) =>  
        (evt.key === 'e' || evt.key === '+' || evt.key === '-') && evt.preventDefault()  
    }  
    onPaste={(evt) => {  
        evt.preventDefault();  
        return false;  
    }}  
```


- Remove the nullable constraint and if the user wants 0 they need to enter 0, yup when using `.typeError('')`
	- Why not use `.required()`? 

	- Use the prev option although a bit weird if you cannot drop the nullable option as it fundamentally changes how the information would be interpreted. (in terms of SMPI it on the UI is represented the same (both are displayed as `-`))
		- OR if the api sends undefined / null which means the UI must support nullables bc the form validation will not allow undefined. If a user was to edit a record with a undefined value that was already saved in the database they would be forced to enter a new valid value.

- write your own custom number input that returns the invalid input to some custom callback like `onValidationError` which you can listen to and manually trigger/register an error on RHF. Then in the Yup schema allow NaN (which would be passed to RHF as an empty string) to be accepted as `null/undefined`. This does two things:
	1. When a nonsensical number is entered `onValidationError` will allow us to register the error while Yup doesn't do anything
	2. When the input is cleared `onValidationError` won't complain and Yup won't do anything except return `null` or `undefined`
(TBC if this option is actually viable haven't actually tried it... i wonder if i could effectively remove the field from the Yup validation schema at that point since its not doing anything... or will TS complain?).

# unable to differentiate between empty vs non-number input value
TL;DR yup schema will transform a non-number and an empty input into `NaN` 

There is no nice way to validate a number AND enforce required. Problem, an empty input (not including defaulting the form value to undefined / null) and a non-number value both translate to NaN. Meaning there is no way to tell in yup if the value is empty (and should trigger required validation) or not a number (and should trigger typeError validation).

Schema with no way to differentiate required and typeError validation: 
```
yup.number()
.transform((value) = > isNaN(value) ? undefined : value)
.typeError('is not a number)
.required('is required')
```

Result: Value is an empty string
- input value: empty, technically '' and not null or undefined 
- ==transform value:  NaN==
- isNan: true
- isNumber: false
Result: Value is a non number
- input value: `fff`
- ==transform value: NaN==
- isNan: true
- isNumber: false
As you can see isNan and isNumber produce the same transformValue so yup can't tell if the input is empty or a non-number.

## what can I do?
- You can only use `required` or `typeError`, and just make the error message reflect both cases e.g. `is required and must be a valid number`
	- Remove non-numbers from being used at the input level, this way we don't need to use typeError and just enforce required/nullable validation.
		- This can be achieved by setting the input type to `number` although be careful of the pitfalls associated with relying on input type number. https://technology.blog.gov.uk/2020/02/24/why-the-gov-uk-design-system-team-changed-the-input-type-for-numbers/ 
- You can use string instead of number. There an empty input will not results in NaN but an '' (empty str). Allowing the capacity to determine if a value is empty or not a number.
	- Unfortunately if you are using typescript this isn't very viable as it means deviating from the model used by the form control (assuming you are using something like Formik or React-Hook-Form)

## other notes
quick console logs to paste into the transform function
```
console.log('value', val);  
console.log('isNan', isNaN(val));  
console.log('isNumber', isNumber(Number(val)));  
console.log('isEmpty', isString(val) && val === '');
```


# dealing with nullable number inputs
Setup
- Making sure the input is a valid number should be done at the input level not the validation schema level. e.g. `<TextField type='number'>`
	- this will lock out most non-number characters (except `.` and `e`)
- Yup schema should say 
```
	foo: yup
		.number()
		.transform((_, val) => val === Number(val) ? val : null))
		.nullable()
```
- The `transform` is doing: transforms the NaN into a `null`, NaN happens when the input is left empty (empty string is converted to NaN) or a non-number is provided (is converted to NaN). 
- This will not work if we are using `required` validation (see unabled to differentiate between empty vs non-number input value)

**Limitations**
- because the input is type `number` we prevent users from entering non-numbers at an input level EXCEPT for `.` and `e`. Its not the end of the world though. As this will be saved as `null` because we transform any NaN's into null. It does mean there are poor UX cases.