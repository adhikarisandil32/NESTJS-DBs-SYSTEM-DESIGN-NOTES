## Error Stack Tracing in Nodejs

- `Error.captureStackTrace(myObj)` puts the captured stack trace (`.stack` property) into the object `myObj`. It is very similar to `new Error(errorMessage).stack`. In simple terms, if you always want the stack of the error, always throw it using `throw new Error()` which will also generate the stack automatically.
