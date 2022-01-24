{#chapter-3}
# Chapter 3: Working effectively in a monorepo

In the previous two chapters we set up a `bookstore` application that renders a list of books for users to purchase.

In this chapter we explore how Nx enables us to work more effectively.

## The dependency graph

As we've seen in [Chapter 1](#chapter-1), Nx automatically generates the dependency graph for us. So why don't we see how it looks now?

```bash
nx dep-graph
```

![Dependency graph of the workspace](images/3-dep-graph.png)

Nx knows the dependency graph of the workspace without us having to configure anything. Because of this ability, Nx also understands which projects within the workspace are affected by any given changeset. Moreover, it can help us verify the correctness of the  affected projects.

I> Note that you can also manually add so-called "implicit dependencies" for those rare cases where there needs to be a dependency which can not be automatically inferred from source code. Read more about that here: https://nx.dev/configuration/projectjson#implicitdependencies

## Only recompute affected projects

Let's say we want to add a **checkout** button to each of the books in the list.

We can update our `Book`, `Books`, and `BooksFeature` components to pass along a new `onAdd` callback prop.

**libs/books/ui/src/lib/book/book.tsx**

```
import styled from 'styled-components';
import { Button } from '@acme/ui';

export interface BookProps {
  book: any;
  // New prop
  onAdd: (book: any) => void;
}

const StyledBook = styled.div`
  display: flex;
  align-items: center;
  border-bottom: 1px solid #ccc;
  &:last-child {
    border-bottom: none;
  }
  > span {
    padding: 1rem 0.5rem;
    margin-right: 0.5rem;
  }
  .title {
    flex: 1;
  }
  .rating {
    color: #999;
  }
  .price {
    color: #478d3c;
  }
`;

export const Book = ({ book, onAdd }: BookProps) => {
  const handleAdd = () => onAdd(book);
  return (
    <StyledBook>
      <span className="title">
        {book.title} by <em>{book.author}</em>
      </span>
      <span className="rating">{book.rating}</span>
      <span className="price">${book.price}</span>
      {/* Add button to UI */}
      <span>
        <Button onClick={handleAdd}>Add to Cart</Button>
      </span>
    </StyledBook>
  );
};

export default Book;
```

**libs/books/ui/src/lib/books/books.tsx**

```typescript
import styled from 'styled-components';
import { Book } from '../book/book';

export interface BooksProps {
  books: any[];
 // New prop
  onAdd: (book: any) => void;
}

const StyledBooks = styled.div`
  border: 1px solid #ccc;
  border-radius: 4px;
`;

export const Books = ({ books, onAdd }: BooksProps) => {
  return (
    <StyledBooks>
      {books.map(book => (
        // Pass down new callback prop
        <Book key={book.id} book={book} onAdd={onAdd} />
      ))}
    </StyledBooks>
  );
};

export default Books;
```

**libs/books/feature/src/lib/books-feature.tsx**

```typescript
import { useEffect, useState } from 'react';
import styled from 'styled-components';
import { getBooks } from '@acme/books/data-access';
import { Books, Book } from '@acme/books/ui';

export const BooksFeature = () => {
  const [books, setBooks] = useState<any[]>([]);

  useEffect(() => {
    getBooks().then(setBooks);
  }, [
    // This effect runs only once on first component render
    // so we declare it as having no dependent state.
  ]);

  return (
    <>
      <h2>Books</h2>
      {/* Pass a stub callback for now */}
      {/* We'll implement this properly in Chapter 4 */}
      <Books books={books} onAdd={book => alert(`Added ${book.title}`)} />
    </>
  );
};

export default BooksFeature;
```

By leveraging the dependency graph, Nx is not only able to understand how the workspace projects relate to each other, but combining this with the Git history, Nx is able to determine which projects were affected by a given changeset.

We can ask Nx to show us how this change *affects* the projects within our workspace using the so-called "affected command".

```bash
nx affected:dep-graph
```

![Affected dependencies](images/3-affected-dep-graph.png)

As we can see, Nx knows that the `books-ui` library has changed starting from the Git `main` branch. Using this information, Nx walks up the dependency graph and highlights all the dependent projects affected by this change in *red*. 

But there is more. We can not only just visualize this change, but we can use various commands to run only against this affected set of projects. Hence, we can just re-test, re-lint or re-build what changed.

```bash
// build only the affected apps
nx affected:build

// run unit tests on affected projects
nx affected:test

// run linting on affected projects
nx affected:lint

// run e2e tests on affected projects
nx affected:e2e
```

Nx topologically sorts the projects such that they are run from bottom to top. That is, projects at the bottom of the dependency chain run first. All these tasks are also parallelized by default (you can customize the amount of parallel tasks using `--maxParallel`).

I> Nx uses 3 parallel tasks by default. You can customize the amount using the `--maxParallel` flag.

All of the `affected:*` commands use the Git history, comparing the current HEAD with a "base" to determine which Nx project(s) got changed. By default "base" refers to the `main` branch. You can customize that by either passing the `--base` flag to the command or by changing the `defaultBase` property in `nx.json`.

A> Note that in these projects, Nx is using [Jest](https://jestjs.io) and [Cypress](https://www.cypress.io/) to run unit and e2e tests respectively. They make writing and running tests are fast and simple as possible. If you're not familiar with them, please read their documentation to learn more.
A>
A> It is possible to use different executors by specifying them in the project's `project.json` configuration file.

So far we haven't been diligent about verifying that our changes are okay, so unsurprisingly our tests are failing.

I'll leave it to you as an exercise to fix the broken unit and e2e tests. A hint for the `App` component test, you should look into the `MemoryRouter` from React Router.

I> **Pro-Tip:** You can run a target for an individual project by issuing `nx [target] [project]` such as `nx test books-ui`, `nx test bookstore`, or `nx e2e bookstore-e2e`. You may also pass the `--watch` flag to re-run tests as soon there is a code change.

For the full solution please see the bookstore example repository: https://github.com/jaysoo/nx-react-book-example.

There are some additional affected commands in Nx.

1. `nx affected:apps` - Lists out all applications affected by the changeset.  
2. `nx affected:libs` - Lists out all libraries affected by the changeset.

The listing of affected applications and libraries can be useful in CI to trigger downstream jobs based on the output.

## Computation Caching

When you heavily adopt a monorepo and the number of projects grows, you need to start thinking about scaling. We have already seen how Nx can cut down the amount of projects to recompute drastically by using the previously mentioned "affected" commands.

Nx goes a step further by also using a computation cache. Basically before running any task, Nx calculates its computation hash. As long as the hash is the same, the output of running the task will be the same.

Let's take the example of running a unit test for an application `app1`. By default the computation hash includes

- All the source files of `app1` and its dependencies
- Relevant global configuration
- Versions of external dependencies
- Runtime values provisioned by the user such as the version of Node
- CLI Command flags

![Calculating the computation cache](images/3-computation-hashing.png)

While this is the default behavior, it can also be customized to more specific needs. For instance, lint checks may only depend on the source code of the project and global configs. Or similarly, builds may depend on the dts files of the compiled libs instead of their source.

Once Nx has the computation hash, it verifies whether that specific hash already exists in its cache. If it does, it replays the task's output in the terminal and restores all possible files in the right folders. From a developers perspective it looks like the task was just run, simply a lot faster.

Try it out by yourself by running unit tests for the `books-feature` project. Run it once and then again to see it being restored from the cache the 2nd time.

```bash
// run unit tests
nx test books-feature

// run them again, they should be restored from the cache
nx test books-feature
```

Every Nx workspace has the computation caching enabled by default. Nx stores the cache locally in the `node_modules/.cache/nx` folder. You can customize which operations get cached as well as the exact location of the cache folder in the `nx.json` file under the `taskRunnerOptions` field.

```json
{
  ...
  "tasksRunnerOptions": {
    "default": {
      "runner": "@nrwl/workspace/tasks-runners/default",
      "options": {
        "cacheableOperations": ["build", "lint", "test", "e2e"]
      }
    }
  },
}
```

You can get even more benefits if this cache is not only local, but remotely distributed. Such functionality can be enabled by using Nx Cloud[^nxcloud].

![Remote caching with Nx Cloud](images/3-cloud-cache.png)

If Nx Cloud is enabled, the local cache folder will be synced with a cloud-hosted, remote counterpart. With the remote cache, other team members and CI agents can read from it too and drastically reduce the required computation time. Learn more on <https://nx.app> and the corresponding Nx Cloud docs at <https://nx.app/docs>.

[^nxcloud]: <https://nx.app>

## Adding the API application

It's time to get more practical again and commit your changes if you haven't done so already: `git add . ; git commit -m 'added checkout button'`.

So far our `bookstore` application does not communicate with a real backend service. Let's create one using the [Express](https://expressjs.com) framework.

We'll need to install the `@nrwl/express` collection first.

```bash
npm install --save-dev @nrwl/express
```

Then we can do a dry run of the express app generator.

```bash
nx g @nrwl/express:app api \
--no-interactive \
--frontend-project=bookstore \
--dryRun
```

![Preview of the file changes](images/3-api-dry-run.png)

Everything looks good so let's run it for real.

```bash
nx g @nrwl/express:app api \
--no-interactive \
--frontend-project=bookstore
```

The `--frontend-project` option will add a proxy configuration to the `bookstore` application such that requests going to `/api/*` will be forwarded to the API

Just like our frontend application, we can use Nx to serve the API.

```bash
nx serve api
```

When we open up `http://localhost:3333/api` we'll be greeted by a friendly message.

```json

{ "message": "Welcome to api!" }
```

Next, let's implement the `/api/books` endpoint so that we can use it in our `books-data-access` library.

**apps/api/src/main.ts**

```typescript
import * as express from 'express';

const app = express();

app.get('/api', (req, res) => {
  res.send({ message: 'Welcome to api!' });
});

app.get('/api/books', (req, res) => {
  const books: any[] = [
    {
      id: 1,
      title: 'The Picture of Dorian Gray ',
      author: 'Oscar Wilde',
      rating: 5,
      price: 9.99
    },
    {
      id: 2,
      title: 'Frankenstein',
      author: 'Mary Wollstonecraft Shelley',
      rating: 4,
      price: 7.95
    },
    {
      id: 3,
      title: 'Jane Eyre',
      author: 'Charlotte Brontë',
      rating: 4.5,
      price: 10.95
    },
    {
      id: 4,
      title: 'Dracula',
      author: 'Bram Stoker',
      rating: 4,
      price: 14.99
    },
    {
      id: 5,
      title: 'Pride and Prejudice',
      author: 'Jane Austen',
      rating: 4.5,
      price: 12.85
    }
  ];
  res.send(books);
});

const port = process.env.port || 3333;
const server = app.listen(port, () => {
  console.log(`Listening at http://localhost:${port}/api`);
});
server.on('error', console.error);
```

Let's update our data-access library to call the proxy endpoint.

**libs/books/data-access/src/lib/books-data-access.ts**

```typescript
export async function getBooks() {
  const data = await fetch('/api/books', {
    headers: {
      'Content-Type': 'application/json'
    }
  });
  return data.json();
}
```

If we restart both applications (`nx serve api` and `nx serve bookstore`; or in a single command `nx run-many --target=serve --projects=api,bookstore`) we'll see that our [bookstore](http://localhost:4200) is still working in the browser. Moreover, we can verify that our `/api/books` endpoint is indeed
being called.

![](images/3-api-verify.png)

Let's commit our changes: `git add . ; git commit -am 'added api app'`.

### Sharing models between frontend and backend

Recall that we previously used the `any` type when working with books data. This is bad practice as it may lead to uncaught type errors in production.

A better idea would be to create a utility library containing some shared models to be used by both the frontend and backend.

```bash
nx g @nrwl/node:lib shared-models --no-interactive
```

**libs/shared-models/src/lib/shared-models.ts**

```typescript
export interface IBook {
  id: number;
  title: string;
  author: string;
  rating: number;
  price: number;
}
```

And now we can update the following five files to use the new model:

**apps/api/src/main.ts**

```typescript
import { IBook } from '@acme/shared-models';
// ...

app.get('/api/books', (req, res) => {
  const books: IBook[] = [
    // ...
  ];
  res.send(books);
});

// ...
```

**libs/books/data-access/src/lib/books-data-access.ts**

```typescript
import { IBook } from '@acme/shared-models';

// Add correct type for the return value
export async function getBooks(): Promise<IBook[]> {
  const data = await fetch('http://localhost:3333/api/books');
  return data.json();
}

```

**libs/books/feature/src/lib/books-feature.tsx**

```typescript
...
import { IBook } from '@acme/shared-models';

export const BooksFeature = () => {
  // Properly type the array
  const [books, setBooks] = useState<IBook[]>([]);

  // ...

  return (
    <>
      <h2>Books</h2>
      <Books books={books} onAdd={book => alert(`Added ${book.title}`)} />
    </>
  );
};

export default BooksFeature;
```

**libs/books/ui/src/lib/books/books.tsx**

```typescript
// ...
import { IBook } from '@acme/shared-models';

// Replace any with IBook
export interface BooksProps {
  books: IBook[];
  onAdd: (book: IBook) => void;
}

// ...

export default Books;
```

**libs/books/ui/src/lib/book/book.tsx**

```typescript
// ...
import { IBook } from '@acme/shared-models';

// Replace any with IBook
export interface BookProps {
  book: IBook;
  onAdd: (book: IBook) => void;
}

// ...

export default Book;
```

![dependency graph with `api` and `shared-models`](images/3-shared-models-graph.png)

By using Nx, we have created a shared model library and refactored both frontend and backend code in about a minute.

Another major benefit of working within a monorepo is that we can check in these changes as a *single commit*: `git add . ; git commit -m 'add shared models'`. The corresponding pull-request with the commit will have the full story, rather than being fragmented amongst multiple pull-requests and repositories.

## Automatic code formatting

One of the easiest ways to waste time as a developer is on code style. We can spend *hours* debating with one another on whether we should use semicolons or not (you should); or whether we should use a comma-first style or not (you should not).

[Prettier](https://prettier.io) was created to stop these endless debates over code style. It is highly opinionated and provides minimal configuration options. Best of all, it can format our code *automatically*. This means that we no longer need to manually fix code to conform to the code style.

Nx workspaces come with Prettier installed from the get-go. With it, we can check the formatting of the workspace, and format workspace code automatically.

```bash
# Checks for format conformance with Prettier.
# Exits with error code when the check fails.
nx format:check

# Formats files with Prettier.
nx format:write
```

***

T> **Key points**
T>
T> Nx understands the dependency graph of projects within our workspace.
T> 
T> We can ask Nx to generate the dependency graph automatically, as well as highlight the parts of the graph that are affected by a given changeset.
T>
T> Nx can retest and rebuild only the affected projects within our workspace.
T>
T> By using a monorepo, related changes in different projects can be in the same changeset (i.e. pull-request), which gives us the full picture of the changes.
T>
T> Nx automatically formats our code for us in an opinionated way using Prettier.

