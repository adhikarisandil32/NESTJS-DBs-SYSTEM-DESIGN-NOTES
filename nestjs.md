# NestJS
- `cli.ts` and `main.ts`. In `main.ts` nest factory creates nest app while in `cli.ts` (for `nestjs-command` module) nest factory creates application context (Read More [Here](https://docs.nestjs.com/standalone-applications)).
  - #### Application Context <br/>
    - What is it? <br/>
An "application context" is a lightweight version of the NestJS app that doesn’t start a web server. It only loads the parts of the app needed to run specific tasks, like CLI commands.
It’s created using `NestFactory.createApplicationContext()` (as seen in `cli.ts`), which sets up the NestJS dependency injection system but skips HTTP-related features.
Think of it as a minimal environment to access your app’s services, modules, or logic without running a full server.

# Docker
- A general way of writing a docker compose that uses environment in docker. Write this in a `docker-compose.yml` file and then go `docker compose up -d`, you'll be ready to use docker
```
services:
  database:
    container_name: '${DATABASE_NAME}_database'
    image: postgres:16-bookworm
    env_file:
      - ./.env
    environment:
      POSTGRES_USER: '${DATABASE_USERNAME}'
      POSTGRES_PASSWORD: '${DATABASE_PASSWORD}'
      POSTGRES_DB: '${DATABASE_NAME}'
    ports:
      - '5439:5432'
```

# Database & ORMs

# Other Backend Services
