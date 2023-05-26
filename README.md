# TypeScript Express API with Jest

## Initialize project and import the imports
* Create a directory for your project and `cd` into it.
* Use [NPM](https://docs.npmjs.com/) to initialize the project `npm init -y`.
* Import dependencies `npm install express`.
* Import dev-dependencies `npm install -D typescript supertest nodemon jest ts-jest ts-node @types/jest @types/supertest`.

## Initialize TypeScript
Let's add TypeScript to our project.
```shell
  npx tsc --init
```
The above command will generate a `tsconfig.json` file.
Modify it like below. A quick note on the `exclude` value, these are files that the build will ignore. Not all of them exist yet.
```json
{
  "exclude": [
    "./coverage",
    "./dist",
    "__test__",
    "jest.config.js"
  ],
  "ts-node": {
    "transpileOnly": true,
    "files": true
  },
  "compilerOptions": {
    "target": "ES2016",
    "module": "CommonJS",
    "rootDir": "./src",
    "moduleResolution": "node",
    "checkJs": true,
    "outDir": "./dist",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noImplicitAny": true,
    "skipLibCheck": true
  }
}
```
## Initialize Jest
We add the Jest testing framework to our project.
```shell
  npx ts-jest config:init
```
The above command will generate a `jest.config.js` file. Modify it with the below, so it works with `ts-jest` (this is what makes jest work with TypeScript).
```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
};
```
## Create a basic Express app with TypeScript
Create a `src` directory with two TypeScript files in it: `app.ts` and `server.ts`. In the `src` directory, add another directory: `routes`. In the `routes` directory add a `user.routes.ts` file.

`app.ts`
```typescript
import express, { Application, Request, Response, NextFunction } from "express";

import { router as userRoutes } from "./routes/user.routes";

const app: Application = express();

app.use("/users", userRoutes);

app.use("/", (req: Request, res: Response, next: NextFunction): void => {
  res.json({ message: "Allo! Catch-all route." });
});

export default app;
```

`server.ts`
```typescript
import app from "./app";

const PORT: Number = 5050;

app.listen(PORT, (): void => console.log(`running on port ${PORT}`));
```

`user.routes.ts`
```typescript
import { Router, Request, Response } from "express";

const router = Router();

router.get("/", (req: Request, res: Response): void => {
  let users = ["Goon", "Tsuki", "Joe"];
  res.status(200).send(users);
});

export { router };
```

## Configure `package.json`
Let's configure our `package.json` to use our new tools! To the `scripts` section add the following:
```json
scripts: {
  "test": "jest --coverage",
  "dev": "nodemon ./src/server.ts",
  "build": "tsc"
}
```

## Making sure our API is working
Now let's be sure we haven't made any mistakes so far. Run the command `npm run dev`.. Open a browser and go to `http://localhost:5050/`. You should be greeted with the welcome message we defined on line 10 of app.js `Allo! Catch-all route.`. Now try our user route `http://localhost:5050/users`, where you should find a list of our users from user.routes.ts `["Goon", "Tsuki", "Joe"]`.

## Writing our tests
Now for the moment we've been waiting for... testing.
In our project add a `__tests__` directory. In that directory we'll duplicate the file structure we made in the `src` directory. Creating a `app.test.ts`, `server.test.ts`, and `routes/user.routes.test.ts`.

Let's write our first test, just to make sure jest is working.

`server.test.ts`
```typescript
describe("Server.ts tests", () => {
  test("Math test", () => {
    expect(2 + 2).toBe(4);
  });
});
```

Now we'll user SuperTest to make a network request test.
`app.test.ts`
```typescript
import request from "supertest";

import app from "../src/app";

describe("Test app.ts", () => {
  test("Catch-all route", async () => {
    const res = await request(app).get("/");
    expect(res.body).toEqual({ message: "Allo! Catch-all route." });
  });
});
```

Now our last test will test our `users` route.
`user.routes.test.ts`
```typescript
import request from "supertest";

import app from "../../src/app";

describe("User routes", () => {
  test("Get all users", async () => {
    const res = await request(app).get("/users");
    expect(res.body).toEqual(["Goon", "Tsuki", "Joe"]);
  });
});
```