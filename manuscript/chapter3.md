{id: chapter-3}
# Chapter 3: Working effectively using Nx

In the previous two chapters we setup a `bookstore` application that renders a list of books for users to purchase.

In this chapter we explore the tools that Nx provides us to enable us to work more effectively and more intelligently.

## Dependency graph

As we've seen in [Chapter 1](#chapter-1), Nx automatically generates our dependency graph for us.

```bash
nx dep-graph
```

![Dependency graph of our workspace](images/3-dep-graph.png)

As we see, Nx knows the dependency graph of your applications and libraries without you having to configure anything.

Because of this, Nx also understands which projects within the workspace are affected by a given changeset.

## Understanding and verifying changes

Let's say we want to update the `font-size` and `font-family` in our global styles.

**libs/ui/src/lib/global-styles/global-styles.tsx**

```tsx
export const GlobalStyles = createGlobalStyle`
  body {
    margin: 0;
    font-size: 15px;
    font-family: -apple-system, BlinkMacSystemFont, Roboto, "Open Sans", "Helvetica Neue", sans-serif
  }

  * {
    box-sizing: border-box;
  }
`;
```

We can ask Nx to show us how this change affects our workspace.

```bash
nx affected:dep-graph
```

![Affected dependencies](images/3-afected-dep-graph.png)

As we see, Nx knows that the `ui` library has changed (since `master`), and indicate all of the projects affected by this change in **red**.

## Automatic code formatting

One of the easiest ways to waste time as a developer is on code styles. We can spend time hours debating with another developer on whether we should use semicolons or not (you should!); or whether we should use a comma-first style or not (you shouldn't).

[Prettier](https://prettier.io) was created to stop these endless debates over code style. It is highly opinionated and provides minimal configuration options.

Best of all, it can format your code automatically! This means you no longer need to manually fix your code to conform to the style.

Nx comes with Prettier out of the box. You can check the formatting of your workspace, or format your workspace files automatically.

```bash
# Checks for format conformance with Prettier.
# Exits with error code when the check fails.
nx format:check

# Formats files with Prettier.
nx format:write
```

The `format:check` command is useful during continuous integration (CI) to make sure developers don't check in badly formatted code.

The `format:write` command is useful when the developer wants to format their code locally.

You'll notice that the above two commands will log out a message saying the affected defaults have been set to `--base=master --head=HEAD`. This is because Nx does not check or format all of your workspace files by default. It intelligently processes only the files that have been changed between `--base` and `--head`, where `base` and `head` refer to git revisions.

The default options are usually what you want, but you can always provide your values if necessary. For example, if your team follows git-flow then you would want to set `--base=develop` when working from the development branch.

You can also force Nx to process *all* files with the `--all` option. Note that this is usually much slower in a large workspace.

## 
