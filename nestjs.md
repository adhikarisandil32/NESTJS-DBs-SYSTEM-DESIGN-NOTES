# NestJS
- `cli.ts` and `main.ts`. In `main.ts` nest factory creates nest app while in `cli.ts` (for `nestjs-command` module) nest factory creates application context (Read More <a target="_blank" href="https://docs.nestjs.com/standalone-applications">Here</a>).
  - #### Application Context <br/>
    - What is it? <br/>
An "application context" is a lightweight version of the NestJS app that doesn’t start a web server. It only loads the parts of the app needed to run specific tasks, like CLI commands.
It’s created using `NestFactory.createApplicationContext()` (as seen in `cli.ts`), which sets up the NestJS dependency injection system but skips HTTP-related features.
Think of it as a minimal environment to access your app’s services, modules, or logic without running a full server.

- Default `nest-commander` uses `CommandFactory` which has its own way of configuring factory. i.e.
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
  `cli.module.ts` being
  ```
    import { Module } from '@nestjs/common';
    import { SeedDatabase } from './db-seed.command';
    import { MyLogger } from '../common-modules/logger.service';
    import { TypeOrmModule } from '@nestjs/typeorm';
    import { DataSource } from 'typeorm';
    import * as dotenv from 'dotenv';
    
    dotenv.config();
    
    // console.log(__dirname)
    @Module({
      imports: [
        // the configuration is same to that of main.ts, this way DataSouce can also be injected without hassle
        TypeOrmModule.forRootAsync({
          useFactory: () => ({
            type: 'postgres',
            username: process.env.DATABASE_USERNAME,
            password: process.env.DATABASE_PASSWORD,
            database: process.env.DATABASE_NAME,
            host: 'localhost',
            port: Number(process.env.DATABASE_PORT),
            entities: [__dirname + './../**/*.entity{.ts,.js}'], // console.log(__dirname) to see where it is looking for entities for seeding
            //  migrations: [__dirname + '/migrations/**/*{.ts,.js}'], // because it is for seeding only, migrations not needed
            synchronize: false,
          }),
          dataSourceFactory: async (options) => {
            const datasource = await new DataSource(options).initialize();
    
            return datasource;
          },
        }),
      ],
      providers: [SeedDatabase, MyLogger],
    })
    export class CliModule {}
  ```
  because of which the command module doesn't recognize the setup for its own application, like path parsing on absolute import. As you can see above, `MyLogger` being imported relatively instead of `src/common-modules/logger.service`. For that, another package named <a href="https://www.npmjs.com/package/nestjs-command" target="_blank">`nestjs-command`</a> is used which utilizes Nestjs's application context because of which nest recognizes its setup for path parsing as well.

- Looking at above, we require the same configuration in `cli.module.ts` and `main.ts` for database, so why don't we export the database only configuration to its own `database.module.ts` file and use whenever it is required for e.g.
  ```
    import { Module } from '@nestjs/common';
    import { TypeOrmModule } from '@nestjs/typeorm';
    import { DataSource } from 'typeorm';
    import * as dotenv from 'dotenv';
    
    dotenv.config();
    
    @Module({
      imports: [
        TypeOrmModule.forRootAsync({
          useFactory: () => ({
            type: 'postgres',
            username: process.env.DATABASE_USERNAME,
            password: process.env.DATABASE_PASSWORD,
            database: process.env.DATABASE_NAME,
            host: 'localhost',
            port: +process.env.DATABASE_PORT,
            entities: [__dirname + './../../**/*.entity{.ts,.js}'],
            migrations: [__dirname + '/migrations/**/*{.ts,.js}'],
            synchronize: false,
          }),
          dataSourceFactory: async (options) => {
            const datasource = await new DataSource(options).initialize();
    
            return datasource;
          },
        }),
      ],
    })
    export class DatabaseModule {}
  ```
  and now the `cli.module.ts` and `main.ts` become more clean, as below
  ```
    // filename -> cli.module.ts
    import { Module } from '@nestjs/common';
    import { MyLogger } from '../common-modules/logger.service';
    import { DatabaseModule } from '../common-modules/database/database.module';
    import { SeedUsersDatabase } from './db-seed.command';
    
    @Module({
      imports: [DatabaseModule],
      providers: [SeedUsersDatabase, MyLogger],
    })
    export class CliModule {}
  ```

- When to use `imports` and `exports` in `@Module` decorator in nest
  ```
    // cats.module.ts
    import { Module } from '@nestjs/common';
    import { CatsService } from './cats.service';
    import { CatsController } from './cats.controller';

    @Module({
      providers: [CatsService],
      controllers: [CatsController],
      exports: [CatsService], // Exports CatsService to other modules
    })
    export class CatsModule {}

    // this is exports
  ```
  ```
    // users.module.ts
    import { Module } from '@nestjs/common';
    import { UsersController } from './users.controller';
    import { CatsModule } from '../cats/cats.module'; // Imports CatsModule

    @Module({
      imports: [CatsModule], // Now UsersModule can use CatsService
      controllers: [UsersController],
    })
    export class UsersModule {}

    // this is imports
  ```
  

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
## TypeORM
- use `@JoinColumn({name: 'user_id'})` decorator to rename the new table that will be created by creating the relation for e.g. `@ManyToOne(() => Users, (user) => user.id, { nullable: false })` on the table `todos`. `@Column` on column that will be on that table only, not on foreign key.
- Always beneficial creating `base.entity.ts` file that any module's entity utilizes as follows:
  ```
  import {
    BaseEntity,
    CreateDateColumn,
    DeleteDateColumn,
    PrimaryGeneratedColumn,
    UpdateDateColumn,
  } from 'typeorm';
  
  export class DBBaseEntity extends BaseEntity {
    @PrimaryGeneratedColumn()
    id: number;
  
    @CreateDateColumn({ type: 'timestamptz', name: 'created_at' })
    createdAt: Date;
  
    @UpdateDateColumn({ type: 'timestamptz', name: 'updated_at' })
    updatedAt: Date;
  
    @DeleteDateColumn({ type: 'timestamptz', name: 'deleted_at' })
    deletedAt: Date;
  }
  ```
  That way now just extend the `DBBaseEntity` on any module's entity that will always contain the `BaseEntity`.

# Others
