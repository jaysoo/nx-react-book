{id: chapter-2}
# Chapter 2: Libraries

We have the skeleton of our application from [Chapter 1](#chapter-1).

So now we can start adding to our application by creating and using libraries.

## Types of libraries

In a workspace, libraries are generally divided into four different types:

**Feature**
: Libraries that implement "smart" UI (e.g. is effectful, is connected to data sources, handles routing, etc.) for specific **business use cases**.
  
**UI**
: Libraries that contain only **presentational** components. That is, components that render purely from their props, and calls function handlers when interaction occurs.
  
**Data-access**
: Libraries that contain the means for interacting with external data services; external services are typically backend services.
  
**Utility**
: Libraries that contain common utilities that are shared by many projects.

Why do we make these distinctions between libraries? Good question! It is good to set boundaries for what a library should and should not do. This demarcation makes it easier to understand the capabilities of each library, and how they interact with each other.

More concretely, we can form rules about what each types of libraries can depend on. For example, UI libraries cannot use feature or data-access libraries, because doing so will mean that they are effectful.

We'll see in [the next chapter](#chapter-3) how we can use Nx to strictly enforce these boundaries.

## The `generate` command

The `nx generate` or the `nx g` command, as it is aliased, allows us to use Nx schematics to create new applications, components, libraries, and more, to our workspace.

## Feature libraries

Let's create our first feature library: `books`.

```bash
nx g lib feature \
--directory books \
--parent-route apps/bookstore/src/app/app.tsx
```

The `--directory` option allows us to group our libraries by nesting them under their parent directory. In this case the library is created in the `libs/books/feature` folder.

The `--parent-route` option lets Nx know that we want to make our feature library to be routable inside the *parent component*. This option is not needed, but it is useful because Nx will do three things for us.

1. Update `apps/bookstore/src/app/app.tsx` with the new route.
2. Update `apps/bookstore/src/main.tsx` to add `BrowserRouter` if it does not exist yet.
3. Add [`react-router-dom`](https://reacttraining.com/react-router/web/guides/quick-start) and related dependencies to the workspace, if necessary.

I> **Pro-tip:** You can pass the `--dry-run` option to `generate` to see the effects of the command before committing to disk.

Once the command completes, you should see the new directory.

```
myorg
├── (...)
├── libs
│   ├── (...)
│   └──products
│       └── feature
│           ├── src
│           │   ├── lib
│           │   └── index.ts
│           ├── .eslintrc
│           ├── jest.config.js
│           ├── README.md
│           ├── tsconfig.app.json
│           ├── tsconfig.json
│           └── tsconfig.spec.json
└── (...)
```

Nx generated our library with some default code as well as scaffolding for linting (ESLint) and testing (Jest). You can run them with:

```bash
nx lint books-feature
nx test books-feature
```

And now we have our library! Wasn't that easy? Something that may have taken minutes (or longer?) now takes only takes a few seconds.

You'll also see that the `App` component for `bookstore` has been updated to include the new route.

```tsx
import React from 'react';
import { Link, Redirect, Route } from 'react-router-dom';

import { BooksFeature } from '@myorg/books/feature';

export const App = () => {
  return (
    <>
      <header>
        <h1>Welcome to the Bookstore!</h1>
        <div role="navigation">
          <ul>
            <li>
              <Link to="/books">Books</Link>
            </li>
          </ul>
        </div>
      </header>
      <Route path="/books" component={BooksFeature} />
      <Route exact path="/" render={() => <Redirect to="/books" />} />
    </>
  );
};

export default App;
```

Additionally, the `main.tsx` file for `bookstore` has also been updated to render `<BrowserRouter />`. This render is needed in order for `<Route />` components to work, and Nx will handle the file update for us if necessary.

```tsx
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom';

import App from './app/app';

ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById('root')
);
```

Restart the development server by running `nx serve bookstore` again and you should see the updated application.

Be aware that when you add a new project to the workspace, you must restart any running development servers. This restart is necessary in order for the TypeScript compiler to pick up new library paths, such as `@myorg/products/feature`.

***

To our despair, when we navigate to <http://localhost:4200> again, we see an ugly looking application.

![](images/2-books-feature.png)

Let's remedy this situation by adding a component library that will provide better styling!

## UI libraries

We can create new UI library as follows.

```bash
nx g lib ui --directory ''
```

Please note that we will make heavy use of [`styled-components`](https://www.styled-components.com) in this component library. Don't fret if you are not familiar with `styled-components`. If you know CSS then you should not have a problem understanding this section. To learn more about `styled-components` you can check our their [documentation](https://www.styled-components.com/docs/basics).


Back to the example. You should have a new folder: `libs/ui`.

```
myorg
├── (...)
├── libs
│   ├── (...)
│   ├── ui
│   │   ├── src
│   │   │   ├── lib
│   │   │   └── index.ts
│   │   ├── .eslintrc
│   │   ├── jest.config.js
│   │   ├── README.md
│   │   ├── tsconfig.app.json
│   │   ├── tsconfig.json
│   │   └── tsconfig.spec.json
└── (...)
```

This library isn't quite useful yet, so let's add in some components.

```bash
nx g component GlobalStyles --project ui --export
nx g component Button --project ui --export
nx g component Header --project ui --export
nx g component Main --project ui --export
nx g component NavigationList --project ui --export
nx g component NavigationItem --project ui --export
```

The `--project` option specifies which project (as found in the `projects` section of `workspace.json`) to add the new component to.

The `--export` option tells Nx to export the new component in the `index.ts` file of the project so that it can be imported elsewhere in the workspace. You may leave this option off if you are generating private/internal components.

If you do forget the `--export` option you can always manually add the export to `index.ts`.

Next, let's go over the implementation of each of the components and what their purposes are.

### `GlobalStyles`

This component injects a global stylesheet into our application when used.

This component is useful for overriding global style rules such as `body { margin: 0 }`.

**libs/ui/src/lib/global-styles/global-styles.tsx**

```tsx
import React from 'react';
import { createGlobalStyle } from 'styled-components';

export const GlobalStyles = createGlobalStyle`
  body {
    margin: 0;
    font-size: 16px;
    font-family: sans-serif;
  }

  * {
    box-sizing: border-box;
  }
`;

export default GlobalStyles;
```

### `Button`

This component is pretty self-explanatory. It renders a styled button and passes through other props to the actual `<button>`.

**libs/ui/src/lib/button/button.tsx**

```tsx
import React, { ButtonHTMLAttributes } from 'react';
import styled from 'styled-components';

const StyledButton = styled.button`
  font-size: 0.8rem;
  padding: 0.5rem;
  border: 1px solid #ccc;
  background-color: #fafafa;
  border-radius: 4px;

  &:hover {
    background-color: #80a8e2;
    border-color: #0e2147;
  }
`;

export const Button = ({
  children,
  ...rest
}: ButtonHTMLAttributes<HTMLButtonElement>) => {
  return <StyledButton {...rest}>{children}</StyledButton>;
};

export default Button;
```

### `Header` and `Main`

These two components are used for layout. The header component forms the top header bar, while the main component takes up the rest of the page.

**libs/ui/src/lib/header/header.tsx**

```tsx
import React, { HTMLAttributes } from 'react';
import styled from 'styled-components';

const StyledHeader = styled.header`
  padding: 1rem;
  background-color: #2657ba;
  color: white;
  display: flex;
  align-items: center;

  a {
    color: white;
    text-decoration: none;

    &:hover {
      text-decoration: underline;
    }
  }
  
  > h1 {
    margin: 0 1rem 0 0;
    padding-right: 1rem;
    border-right: 1px solid white;
  }
`;

export const Header = (props: HTMLAttributes<HTMLElement>) => (
  <StyledHeader>{props.children}</StyledHeader>
);

export default Header;
```

**libs/ui/src/lib/main/main.tsx**

```tsx
import React, { HTMLAttributes } from 'react';
import styled from 'styled-components';

const StyledMain = styled.main`
  padding: 0 1rem;
  width: 100%;
  max-width: 960px;
`;

export const Main = (props: HTMLAttributes<HTMLElement>) => (
  <StyledMain>{props.children}</StyledMain>
);

export default Main;
```

### `NavigationList` and `NavigationItem`

And finally, the `NavigationList` and `NavigationItem` components will render the navigation bar inside our top `Header` component.

**libs/ui/src/lib/navigation-list/navigation-list.tsx**

```tsx
import React, { HTMLAttributes } from 'react';
import styled from 'styled-components';

const StyledNavigationList = styled.div`
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }
`;

export const NavigationList = (props: HTMLAttributes<HTMLElement>) => {
  return (
    <StyledNavigationList role="navigation">
      <ul>{props.children}</ul>
    </StyledNavigationList>
  );
};

export default NavigationList;
```

**libs/ui/src/lib/navigation-item/navigation-item.tsx**

```tsx
import React, { LiHTMLAttributes } from 'react';
import styled from 'styled-components';

const StyledNavigationItem = styled.li`
  margin-right: 1rem;
`;

export const NavigationItem = (props: LiHTMLAttributes<HTMLLIElement>) => {
  return <StyledNavigationItem>{props.children}</StyledNavigationItem>;
};

export default NavigationItem;
```

## Using the UI library

Now we can use the new library in our `bookstore`'s app component.

**apps/bookstore/src/app/app.tsx**

```tsx
import React from 'react';
import { Link, Redirect, Route } from 'react-router-dom';

import { BooksFeature } from '@myorg/books/feature';
import {
  GlobalStyles,
  Header,
  Main,
  NavigationItem,
  NavigationList
} from '@myorg/ui';

export const App = () => {
  return (
    <>
      <GlobalStyles />
      <Header>
        <h1>Bookstore</h1>
        <NavigationList>
          <NavigationItem>
            <Link to="/books">Books</Link>
          </NavigationItem>
        </NavigationList>
      </Header>
      <Main>
        <Route path="/books" component={BooksFeature} />
        <Route exact path="/" render={() => <Redirect to="/books" />} />
      </Main>
    </>
  );
};

export default App;
```

Finally, let's restart our server (`nx serve bookstore`) and we will see a much improved UI.

![](images/2-styling.png)

We'll save our progress with a new commit.

```bash
git add .
git commit -m 'Add products feature and ui libraries'
```

That's great, but we are still not seeing any products, so let's do something about this.

## Data-access libraries

What we want to do is fetch data from *somewhere* and display that in our products feature. Since we will be calling a backend service we should create a new **data-access** library.

```bash
nx g @nrwl/web:lib data-access --directory products
``` 

You may have noticed that we are using a prefix `@nrwl/web:lib` instead of just `lib` like in our previous examples. This `@nrwl/web:lib` syntax means that we want Nx to run the `lib` (or `library`) schematic provided by the `@nrwl/web` collection.

We were able to go without this prefix previously because the `workspace.json` configuration has set `@nrwl/react` as the default option.

```json
{
  // ...
  "cli": {
    "defaultCollection": "@nrwl/react"
  },
  // ...
}
```

In this case, the `@nrwl/web:lib` schematic will create a library to be used in a web (i.e. browser) context without assuming the framework used. In contrast, when using `@nrwl/react:lib`, it assumes that you want to generate a default component as well as potentially setting up routes.

Back to the example. Let's modify the library to export a `getBooks` function to load our list of books.

**libs/products/data-access/src/lib/products-data-access.ts**

```typescript
export async function getBooks(): Promise<any[]> {
  // TODO: We'll wire this up to an actual API later.
  // For now we are just returning some fixtures.
  return [
    {
      id: 1,
      name: 'The Picture of Dorian Gray ',
      author: 'Oscar Wilde',
      rating: 5,
      price: 9.99
    },
    {
      id: 2,
      name: 'Frankenstein',
      author: 'Mary Wollstonecraft Shelley',
      rating: 4,
      price: 7.95
    },
    {
      id: 3,
      name: 'Jane Eyre',
      author: 'Charlotte Brontë',
      rating: 4.5,
      price: 10.95
    },
    {
      id: 4,
      name: 'Dracula',
      author: 'Bram Stoker',
      rating: 4,
      price: 14.99
    },
    {
      id: 5,
      name: 'Pride and Prejudice',
      author: 'Jane Austen',
      rating: 4.5,
      price: 12.85
    }
  ];
}
```

### Using the data-access library

The next step is to use the `getBooks` function within our `books` feature. We can do this with React's `useEffect` and `useState` hooks.

**libs/books/feature/src/lib/books-feature/books-feature.tsx**

```tsx
import React, { useEffect, useState } from 'react';
import styled from 'styled-components';
import { getBooks } from '@myorg/products/data-access';
import { Books, Book } from '@myorg/products/ui';

export const BooksFeature = () => {
  const [books, setBooks] = useState([]);

  useEffect(() => {
    getBooks().then(setBooks);
  }, [
    // This effect runs only once on first component render
    // so we declare it as having no dependent state.
  ]);

  return (
    <>
      <h2>Books</h2>
      <Books>
        {books.map(book => (
          <Books books={books} onAdd={book => alert(`Added ${book.name}`)} />
        ))}
      </Books>
    </>
  );
};

export default BooksFeature;
```

You'll notice that we're using two new components: `Books` and `Book`. They can be created as follows.

```bash
nx g lib ui --directory products
nx g component Books --project products-ui --export
nx g component Book --project products-ui
```

We generally want to put *presentational* components into their own UI library. This will prevent effects from bleeding into them, thus making them easier to understand and test.

Again, we will see in [Chapter 3](#chapter-3) how Nx enforces module boundaries.
 
**libs/products/ui/src/lib/books/books.tsx**
```tsx
import React from 'react';
import styled from 'styled-components';
import { Book } from '../book/book';

export interface BooksProps {
  books: any[];
}

const StyledBooks = styled.div`
  border: 1px solid #ccc;
  border-radius: 4px;
`;

export const Books = ({ books }: BooksProps) => {
  return (
    <StyledBooks>
      {books.map(book => (
        <Book key={book.id} book={book} />
      ))}
    </StyledBooks>
  );
};

export default Books;
```

**libs/products/ui/src/lib/book/book.tsx**

```tsx
import React from 'react';
import styled from 'styled-components';
import { Button } from '@myorg/ui';

export interface BookProps {
  book: any;
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
  return (
    <StyledBook>
      <span className="title">
        {book.name} by <em>{book.author}</em>
      </span>
      <span className="rating">{book.rating}</span>
      <span className="price">${book.price}</span>
      <span>
        <Button onClick={() => onAdd(book)}>Add To Cart</Button>
      </span>
    </StyledBook>
  );
};

export default Book;
```

Restart the server to check out our feature in action.

![](images/2-data-access.png)

That's great and all, but you may have observed a couple of problems.

1. The `getProducts` data-access function is a stub and doesn't actually call out to a backend service.
   
2. We've been using `any` types when dealing with books data. For example, the return type of `getProducts` is `any[]` and our `BookProp` takes specifies `{ book: any }`. This makes our code unsafe and can lead to production bugs.
   
We'll address both problems in the [next chapter](#chapter-3).

***

T> **Key points**
T>
T> Libraries are separated into four types: *feature*, *UI*, *data-access*, and *util*.
T> 
T> Nx provides us with the `nx generate` or `nx g` command to *quickyl* create new libraries from scratch.
T>
T> When running `nx g` we can optionally provide a collection such as `@nrwl/web:lib` as opposed to `lib`. This will tell Nx to use the schematic from that specific collection rather than taking the workspace's `defaultCollection`. 
