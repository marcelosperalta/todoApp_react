<div align="center">
<img height="200" src="./img_mern.png"/>
&nbsp;&nbsp;&nbsp;&nbsp;
<img height="200" src="https://raw.githubusercontent.com/github/explore/80688e429a7d4ef2fca1e82350fe8e3517d3494d/topics/typescript/typescript.png"/>
</div>

# :white_check_mark: To-do App

A To-do App with [MongoDB](https://www.mongodb.com/), [Express](https://expressjs.com/), [React](https://reactjs.org/), [Node.js](https://nodejs.org/en/), and [TypeScript](https://www.typescriptlang.org/).

## :star: Source

[How to Build a Todo App with React, TypeScript, NodeJS, and MongoDB](https://www.freecodecamp.org/news/how-to-build-a-todo-app-with-react-typescript-nodejs-and-mongodb/) by [Ibrahima Ndaw](https://github.com/ibrahima92) on [freeCodeCamp.org](https://www.freecodecamp.org/)

## :cloud: Getting Started **Server-side**

### :dvd: Generate the ```tsconfig.json```

```
yarn init -y
```

### :heavy_minus_sign: Structure of the project

```
├── server
    ├── dist
    ├── node_modules
    ├── src
        ├── controllers
        |  └── todos
        |     └── index.ts
        ├── models
        |  └── todo.ts
        ├── routes
        |  └── index.ts
        └── types
           └── todo.ts
        ├── app.ts
    ├── nodemon.json
    ├── package.json
    ├── tsconfig.json
```

![server structure](./img_screenshot02.png)

### ⚙ Configuring TypeScript with [tsconfig](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) using [tsc](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-6.html#--init-command-line-option)

```
tsc --init
```

Delete the ```tsconfig.json``` original settings and paste the text below:

```
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "outDir": "dist/js",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["src/types/*.ts", "node_modules", ".vscode"]
}
```

:black_small_square: ```outDir:``` tells the compiler to put the compiled code into the dist/js folder.  
:black_small_square: ```rootDir:``` informs TypeScript to compile every .ts file located in the src folder.  

:black_small_square: ```include:``` tells the compiler to include files that are in the src directory and sub-directory.  
:black_small_square: ```exclude:``` will exclude the files or folders passed in the array during compile-time.  

![tsconfig.json](./img_screenshot03.png)

### :dvd: Install the dependencies to enable [TypeScript](https://www.typescriptlang.org/)  

```
yarn add typescript -g
```

### :dvd: Install the dependencies [Express](https://expressjs.com/), [CORS](https://github.com/expressjs/cors), and [Mongoose](https://mongoosejs.com/) to use [Express](https://expressjs.com/) and [MongoDB](https://www.mongodb.com/)  

```
yarn add express cors mongoose
```

:heavy_exclamation_mark: install their types as development dependencies to help the TypeScript compiler understand the packages.

:loudspeaker: see _[type declarations](https://www.typescriptlang.org/docs/handbook/declaration-files/consumption.html)_

```
yarn add -D @types/node @types/express @types/mongoose @types/cors
```

![package.json](./img_screenshot04.png)

### :dvd: Install the dependencies [Concurrently](https://github.com/kimmobrunfeldt/concurrently#readme), and [nodemon](https://nodemon.io/)  


[Concurrently](https://github.com/kimmobrunfeldt/concurrently#readme) will help compile the TypeScript code, keep watching for changes, and also start the server simultaneously.

```
yarn add -D concurrently nodemon
```

:heavy_exclamation_mark: update the ```package.json```

```
"scripts": {
  "build": "tsc",
  "start": "concurrently \"tsc -w\" \"nodemon dist/js/app.js\""
}
```

![package.json](./img_screenshot05.png)

### :hash::zero::one: Create a Todo Type

:open_file_folder: [server/src/types/todo.ts](https://github.com/marcelosperalta/todoApp_react/blob/master/server/src/types/todo.ts)

```
import { Document } from "mongoose"

export interface ITodo extends Document {
  name: string
  description: string
  status: boolean
}
```

### :hash::zero::two: Create a Todo Model

:open_file_folder: [server/src/models/todo.ts](https://github.com/marcelosperalta/todoApp_react/blob/master/server/src/models/todo.ts)

```
import { ITodo } from "./../types/todo"
import { model, Schema } from "mongoose"

const todoSchema: Schema = new Schema(
  {
    name: {
      type: String,
      required: true,
    },

    description: {
      type: String,
      required: true,
    },

    status: {
      type: Boolean,
      required: true,
    },
  },
  { timestamps: true }
)

export default model<ITodo>("Todo", todoSchema)
```
### :hash::zero::three: Create API controllers
**Get, Add, Update and Delete Todos**

:open_file_folder: [server/src/controllers/todos/index.ts](https://github.com/marcelosperalta/todoApp_react/blob/master/server/src/controllers/todos/index.ts)

```
import { Response, Request } from "express"
import { ITodo } from "./../../types/todo"
import Todo from "../../models/todo"

// GET
const getTodos = async (req: Request, res: Response): Promise<void> => {
  try {
    const todos: ITodo[] = await Todo.find()
    res.status(200).json({ todos })
  } catch (error) {
    throw error
  }
}

// POST
const addTodo = async (req: Request, res: Response): Promise<void> => {
  try {
    const body = req.body as Pick<ITodo, "name" | "description" | "status">

    const todo: ITodo = new Todo({
      name: body.name,
      description: body.description,
      status: body.status,
    })

    const newTodo: ITodo = await todo.save()
    const allTodos: ITodo[] = await Todo.find()

    res
      .status(201)
      .json({ message: "Todo added", todo: newTodo, todos: allTodos })
  } catch (error) {
    throw error
  }
}

// PUT
const updateTodo = async (req: Request, res: Response): Promise<void> => {
  try {
    const {
      params: { id },
      body,
    } = req
    const updateTodo: ITodo | null = await Todo.findByIdAndUpdate(
      { _id: id },
      body
    )
    const allTodos: ITodo[] = await Todo.find()
    res.status(200).json({
      message: "Todo updated",
      todo: updateTodo,
      todos: allTodos,
    })
  } catch (error) {
    throw error
  }
}

// DELETE
const deleteTodo = async (req: Request, res: Response): Promise<void> => {
  try {
    const deletedTodo: ITodo | null = await Todo.findByIdAndRemove(
      req.params.id
    )
    const allTodos: ITodo[] = await Todo.find()
    res.status(200).json({
      message: "Todo deleted",
      todo: deletedTodo,
      todos: allTodos,
    })
  } catch (error) {
    throw error
  }
}

export { getTodos, addTodo, updateTodo, deleteTodo }
```

### :hash::zero::four: Create API routes

:open_file_folder: [server/src/routes/index.ts](https://github.com/marcelosperalta/todoApp_react/blob/master/server/src/routes/index.ts)

```
import { Router } from "express"
import { getTodos, addTodo, updateTodo, deleteTodo } from "../controllers/todos"

const router: Router = Router()

router.get("/todos", getTodos)

router.post("/add-todo", addTodo)

router.put("/edit-todo/:id", updateTodo)

router.delete("/delete-todo/:id", deleteTodo)

export default router
```

### :hash::zero::five: Create a Server  

:page_with_curl: Create a ```nodemon.json``` file to hold the MongoDB credentials.

```
{
    "env": {
        "MONGO_USER": "your-username",
        "MONGO_PASSWORD": "your-password",
        "MONGO_DB": "your-db-name"
    }
}
```

:rotating_light: add the ```nodemon.json``` to your [.gitignore](https://git-scm.com/docs/gitignore) file to protect your DB access data.

![nodemon.json](./img_screenshot06.png)

:loudspeaker: you can get the credentials by [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)

:page_with_curl: Create a ```app.ts``` file.

:open_file_folder: [server/src/app.ts](https://github.com/marcelosperalta/todoApp_react/blob/master/server/src/app.ts)

## :computer: **Client-side** with React and TypeScript

#### ⚙ Setting up

:black_small_square: Create a new [React App](https://reactjs.org/docs/create-a-new-react-app.html) adding [TypeScript](https://create-react-app.dev/docs/adding-typescript/)

:rotating_light: the ```client``` folder needs to be at the same level of the ```server``` folder

```
npx create-react-app client --template typescript
```

:black_small_square: open the ```client``` folder

```
cd client
```

:black_small_square: Install the [Axios](https://github.com/axios/axios#axios) library to be able to fetch remote data.

```
yarn add axios
```