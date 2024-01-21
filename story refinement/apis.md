- were changing an existing implementation to use a new API for 1 customer, but we support other customers to, will they be impacted by this change? 

- since were going for a bulk update where multiple items are being updated, what happens if that fails? do we know which items will need to be retried? can we handle it gracefully? (apparently the error response sucks)

- Is this new api async so when we hit it we don't get blocked until its done? 

- Is the UPDATE/UPSERT going to be PATCH or PUT or POST?

- When does the operation complete / return a response, is it when the action is executed or when it is complete? aka is this async?

- What does the create path look like vs the update path are their variations that exist?

- What information is required to perform a DELETE / UPDATE?