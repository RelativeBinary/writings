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