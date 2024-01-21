- Can we use the design systems spacing variables instead of numbers?
- Using module css for identifiers must be an imported value 
```
<div id={styles['the-style']} />

// not 
<div id={'the-style'} />
```

- There is a selector specificity ranking that determines which style wins out, if you have issues you can check that in dev tools by hovering over css classes to see ther selector specificity ranking.
