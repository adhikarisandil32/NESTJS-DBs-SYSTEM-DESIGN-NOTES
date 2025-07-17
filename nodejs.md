## Error Stack Tracing in Nodejs

- `Error.captureStackTrace(myObj)` puts the captured stack trace (`.stack` property) into the object `myObj`. It is very similar to `new Error(errorMessage).stack`. In simple terms, if you always want the stack of the error, always throw it using `throw new Error()` which will also generate the stack automatically. [For Reference](https://nodejs.org/api/errors.html#new-errormessage-options). \n General user case of stack tracing as we extend the `Error` class in node.js,
  ```
  class ApiError extends Error {
      constructor(
          statusCode,
          message= "Something went wrong",
          errors = [],
          stack = ""
      ){
          super(message)
          this.statusCode = statusCode
          this.data = null
          this.message = message
          this.success = false;
          this.errors = errors
  
          if (stack) {
              this.stack = stack
          } else{
              Error.captureStackTrace(this, this.constructor)
          }
  
      }
  }
  
  export {ApiError}
  ```
