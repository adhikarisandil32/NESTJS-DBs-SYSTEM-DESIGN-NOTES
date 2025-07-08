# NestJS
- `cli.ts` and `main.ts`. In `main.ts` nest factory creates nest app while in `cli.ts` (for `nestjs-command` module) nest factory creates application context (Read More [Here](https://docs.nestjs.com/standalone-applications)).
  - #### Application Context <br/>
    - What is it? <br/>
An "application context" is a lightweight version of the NestJS app that doesn’t start a web server. It only loads the parts of the app needed to run specific tasks, like CLI commands.
It’s created using `NestFactory.createApplicationContext()` (as seen in `cli.ts`), which sets up the NestJS dependency injection system but skips HTTP-related features.
Think of it as a minimal environment to access your app’s services, modules, or logic without running a full server.

- Default `nest-commander` uses `CommandFactory` has its own way of configuring factory. i.e.
  ```
    import { CommandFactory } from 'nest-commander';
    import { CliModule } from './commands/cli.module';
    
    async function bootstrap() {
      // await CommandFactory.run(AppModule);
    
      // or, if you only want to print Nest's warnings and errors
      await CommandFactory.run(CliModule, ['warn', 'error']);
    }
    
    bootstrap();
  ```
  because of which the command module doesn't recognize the setup for its own application, like path parsing on absolute import. For that, a another package named `nestjs-command` is used which utilizes Nestjs's application context because of which nest recognizes its setup for path parsing as well.

# Docker
- A general way of writing a docker compose that uses environment in docker. Write this in a `docker-compose.yml` file and then go `docker compose up -d`, you'll be ready to use database in docker
```
volumes:
  ${DATABASE_NAME}_pg_data:
    name: '${DATABASE_NAME}_pg_data'

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
    volumes:
      - ../${DATABASE_NAME}_pg_data:/var/lib/postgresql/data
      # - ../data/postgres/initdb.d:/docker-entrypoint-initdb.d
```
- `docker exec -it <container_name> /bin/bash`, with this you can enter inside the bash of the container of the database. To use with user `postgres`, go `su postgres` and then `psql`. You'll be connected to the database of the docker. You can also list all the environment vairables used on the container by using `env` or `printenv` command from the terminal of `root` user. `PGDATA` and `PATH` variables can be important for volume maping in which `PGDATA` represents the path to postgres data.
```
    volumes:
      - ../${DATABASE_NAME}_pg_data:/var/lib/postgresql/data
      # - ../data/postgres/initdb.d:/docker-entrypoint-initdb.d
```

# Database & ORMs
- use `@JoinColumn({name: 'user_id'})` decorator to rename the new table that will be created by creating the relation for e.g. `@ManyToOne(() => Users, (user) => user.id, { nullable: false })` on the table `todos`. `@Column` on column that will be on that table only, not on foreign key.

# Others
