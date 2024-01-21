instead of using an if condition to return a data collection or another data collection. You can try a ternary

```
// dont
if (condition A) {
	return {fooData, size: fooData.length()}
} else {
	cosnt result = fooData.extraThing();
	return {fooData: result, size: result.length()}
}


// do 
const result = condition A ? fooData : fooData.extraThing();

return {fooData: result, size: result.length()}
```