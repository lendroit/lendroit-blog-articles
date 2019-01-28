# NestJS CQRS EventSourcing - The definitive tutorial

This is the first article of a serie about the NodeJS framework NestJS and two backend concepts: CQRS and Event Sourcing.

This was first written as a [Trello board](https://trello.com/b/yrG5pLaO/event-sourcing-cqrs) for my company [BAM](bam.tech) (with images of prodigy, don't ask...). You can follow the tutorial through the tickets of the board, or through this article.

## Motivation

This tutorial is made for you if:

- You know the basics of NodeJS
- You are interested in backend design patterns
- You have heard of CQRS and Event Sourcing and want a first basic implementation of it

## Summary

The series contain 3 parts:

- Create a blog API with NestJS "The old ways": How to create a controller, a service, connect a DB and use TypeORM for a CRUD on an `Article` entity.
- Implement CQRS: Create a command bus and separate the ways the api writes and reads data.
- Implement Event Sourcing: Create the `Event` entity, store the event and rebuild an `Article` from related Events.

For feedback, please open issues on [this repo](https://github.com/lendroit/lendroit-blog-articles)

The tutorial was done using node 8.12, VSCode and Insomnia.

## Setup

### AADev, I have a Hello World on my server

- Install nest: `npm i -g @nestjs/cli`
- Start your project: `nest new blog-croute`
- Init versionning: `cd blog-croute && git init`
- Open your code editor: `code .`
- Start the server: `npm run start:dev`
- Go to http://localhost:3000
- Commit ! (add a .gitignore file with /node_modules first 😙)

### AADev, I have an incredible DX

If you are using VSCode here is a config for debugging:

```JavaScript
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach",
      "port": 9229,
      "timeout": 10000,
      "stopOnEntry": false,
      "restart": true
    }
  ]
}
```

- Run npm run start:debug
- Press F5 (or the play button in the debugger)
- Go to the URL, check that you see "Hello world"
- Put a debugger in the controller right before returning return this.appService.root();
- Add /dist to the .gitignore
- commit ! [solution](https://github.com/Samox/dojo-event-sourcing-cqrs/commit/7f34e3d4718300b242daef5099c2cdfa1112c532)

### AADev, I can read a static article

In your code editor, you see files. Lots of them.
Let go directly into the src folder and look at the controller. This is the entry point of your program. In other words, the controller is the interface between the consumer (a web application for instance) and your program.

In the REST paradigm, the routes like **POST /articles** and and **GET /articles/id** will be handled by the controller.

Now take a look at the get request. There is a decorator, you see that the function will return a string.

Let us create a new route !
It will be a request **GET /articles** that returns an article.

- In the controller.ts add a @Get("/articles") decorator
- Under the decorator, declare a function that takes no argument getArticle() oh but what does it return ?
- Create a file article.dto.ts
- Inside the file we will declare and export the interface ArticleDTO of our articles. It should have a name: string and a content: string. You don't know how to create a interface in TS ?
- Back in the controller. We can now specify what the function will return and return a static article: return { name: 'TS > Flow 😶', content: 'Croute' };
- Check that http://localhost:3000/articles returns your article
- Commit !!! [Solution](https://github.com/Samox/dojo-event-sourcing-cqrs/commit/8cc2f9cf0e32ac489cf5f00dd9feb0ea70290034)

### AADev, I can store an article

We now need a little bit of infrastructure to continue. Let us create a database. _This can be a long process, I'm working on a dockerfile for the infrastructure with the database_.

Then we'll create an entity Article that will represent our model. With NestJs comes an ORM named **TypeORM**.

We will use TypeORM to generate the database migrations and the connection to the database.

After that, we'll plug the controller with the model to be able to write an article.

- Restart your app with npm run start:dev
- Install psql: brew install postgresql
- Check that the database is up: brew services list
- Connect to your database: psql
- Create a new database: createdb croutedb
- Install the orm and the driver: npm install --save @nestjs/typeorm typeorm pg
- Connect our server to the database: replace mysql with postgres, the port is 5432, and the dbname is croutedb if you have copy pasted the code without thinking
- Commit ! [Solution](https://github.com/Samox/dojo-event-sourcing-cqrs/commit/71902e6b7542cc2a5e1a59099d831661c8ea2b24)
- Create a file article.entity.ts
- Inside this file declare a class Article with the decorator @Entity() from typeorm
- In the class, declare an id with the decorator @PrimaryGeneratedColumn('uuid'), the type of id will be a string
- In the class declare a name and a content with the decorator @column()
- Now go to your database psql, \$ \connect croutedb and check that there is table article
- Commit ! Solution Samox/dojo-event-sourcing-cqrs: 71902e6
- Add a new service in the app.service.ts file, storeArticle, it will be async, takes a parameter articleDto, returns a promise<article> (not the DTO, this article has an ID, read this)
- Add a constructor in the service constructor( @InjectRepository(Article) private readonly articleRepository: Repository<Article>,) {}
- At this point your program fails, because it cannot build the Service class, we need to add the TypeORM repository to the module. Let's do it by adding TypeOrmModule.forFeature([Article]) to the imports of the module. (right after the TypeOrmModule.forRootdeclaration)
- In the service complete the storeArticle method: create an article with the method this.articleRepository.create, it takes the dto as a parameter. then save the article with the save method, then return the created article.
- In the controller create a new method with the @Post('articles') decorator. It will return a Promise<ArticleDTO>
- The function shall be called createArticle, it takes one parameter @Body() createArticleDto: ArticleDTO
- Complete the function by calling the service method storeArticle with the createArticleDto and return the result
- Check your route with postman
- Commit !!!! [Solution](https://github.com/Samox/dojo-event-sourcing-cqrs/commit/320d370fa088d8569cb7a678141b18f0d63ce8e4)

### AADev, I can get my articles

Okkkaayyyy, that was the hard part, now that we have created our Entity, our Service and our Controller method, let's return our articles.

Noticed how we have not use the Repository in our controller ? That is the separation of concerns, our Controller consumes a Service. The Service will decide if it uses a repository or anything else, when we'll change the way we store/read our data, the controller shall not be impacted 🤗.

- In the service, create a method getAllArticles, it takes no parameter, it return a promise of array of article Promise<Article[]>
- In the body of the function, call the repository method find() and return the result.
- In the controller, change the get('articles') method, so it return an promise of array of article, and call the service getAllArticle method. (rename the method getArticles, with an s)
- Boom ! Done ! Commit !! [Solution](https://github.com/Samox/dojo-event-sourcing-cqrs/commit/77623c40ac9a86e5859c16598502f78a4e1e651c)

### AADev, I can get one article with the ID

You are on your own on this one. You can do it 🤓.

- tips: the route is 'articles/:id'
- tips2: Read the doc of controllers to understand how to get the param from the route.
- tips3: Use the findOne method of the repository.

- Commit !!!!! [Solution](https://github.com/Samox/dojo-event-sourcing-cqrs/commit/500fbf7ccea5b936e86830158c4d92dcf88a5bd4)
