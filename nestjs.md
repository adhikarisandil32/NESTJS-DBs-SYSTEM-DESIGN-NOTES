# NestJS
- `cli.ts` and `main.ts`. In `main.ts` nest factory creates nest app while in `cli.ts` (for `nestjs-command` module) nest factory creates application context (Read More [Here](https://docs.nestjs.com/standalone-applications)).
  - #### Application Context <br/>
    - What is it? <br/>
An "application context" is a lightweight version of the NestJS app that doesn’t start a web server. It only loads the parts of the app needed to run specific tasks, like CLI commands.
It’s created using NestFactory.createApplicationContext() (as seen in cli.ts), which sets up the NestJS dependency injection system but skips HTTP-related features.
Think of it as a minimal environment to access your app’s services, modules, or logic without running a full server.

# Database & ORMs

# Other Backend Services
