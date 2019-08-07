{id: chapter-3}
# Chapter 3: Working effectively using Nx

In the previous two chapters we setup a `bookstore` application that renders a list of books for users to purchase.

In this chapter we explore the tools that Nx provides us with to enable us to work more effectively.

## The dependency graph

As we've seen in [Chapter 1](#chapter-1), Nx automatically generates the dependency graph for us. So why don't we see how it looks now?

```bash
nx dep-graph
```

![Dependency graph of the workspace](images/3-dep-graph.png)

Nx knows the dependency graph of the workspace without us having to configure anything. Because of this ability, Nx also understands which projects within the workspace are affected by any given changeset. Moreover, it can help us verify that the affected projects are okay.

## Understanding and verifying changes

Let's say we want to update `font-family` in our global styles.

**libs/ui/src/lib/global-styles/global-styles.tsx**

```tsx
// ...
body {
  // Use system fonts from various platforms.
  font-family: -apple-system,system-ui,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Arial,sans-serif;
}
// ...
```

We can ask Nx to show us how this change *affects* the projects within our workspace.

```bash
nx affected:dep-graph
```

![Affected dependencies](images/3-affected-dep-graph.png)

As we can see, Nx knows that the `ui` library has changed from the `master` branch, and has indicated all of the dependent projects affected by this change in *red*. Furthermore, Nx also allows us to retest only the affected projects.

We can **lint** (ESLint) the projects affected by the changeset. 

```bash
nx affected:lint --parallel
```

Or run **unit tests** (Jest) for the affected projects.

```bash
nx affected:test --parallel
```

Or even run **e2e tests** (Cypress) for the affected projects.

```bash
nx affected:e2e --parallel
```

Nx topologically sorts the projects so they are run from bottom to top. That is, projects at the bottom of the dependency chain are run first. We're also using the `--parallel` option to enable Nx to run our projects in parallel.

And just as with the `affected:dep-graph` command, the default base is the `master` branch. This default branch should be correct most of the time, but can be changed with the `--base=[branch]` option. For example, if your team uses git-flow then you may want to use `--base=develop` when doing work on the development branch.

Note that Nx uses  [Jest](https://jestjs.io) and [Cypress](https://www.cypress.io/) to run unit and e2e tests respectively. They make writing and running tests are fast and simple as possible. If you're not familiar with them, please read their documentation to learn more.

So far we haven't been diligent on verifying that our changes are okay, so it isn't surprising that our tests are failing.

![`nx affected:test` failed](images/3-failed-test.png)

![`nx affected:e2e` failed](images/3-failed-e2e.png)

I'll leave it to you as an exercise to fix the unit and e2e tests. **Tip:** Run the tests in watch mode by passing the `--watch` option so that tests are rerun whenever the source or test code change.

There are three additional affected commands in Nx.

1. `nx affected:build` - Builds only the affected apps. We'll go over build and deployment in [Chapter 5](#chapter-5).
2. `nx affected:apps` - Lists out all applications affected by the changeset.  
3. `nx affected:libs` - Lists out all libraries affected by the changeset.

The listing of affected applications and libraries can be useful in CI to trigger downstream jobs based on the output.

And that about does it for affected commands. Next, we'll discuss everyone's favorite topic: *code style*.

## Automatic code formatting

One of the easiest ways to waste time as a developer is on **code styles**. We can spend time *hours* debating with one another on whether we should use semicolons or not -- you should; or whether we should use a comma-first style or not -- you shouldn't.

[Prettier](https://prettier.io) was created to stop these endless debates over code style. It is highly opinionated and provides minimal configuration options. And best of all, it can format our code *automatically*! This means that we no longer need to manually fix code to conform to the code style.

Nx comes prepackaged with Prettier. With it, we can check the formatting of the workspace, and format workspace code automatically.

```bash
# Checks for format conformance with Prettier.
# Exits with error code when the check fails.
nx format:check

# Formats files with Prettier.
nx format:write
```

Lastly, you may want to set up a pre-commit git hook to run `nx format:write` so we can ensure 100% conformance whenever code is checked in. For more details please refer to [Appendix C](#appendix-c). 

***

T> **Key points**
T>
T> Nx understands the dependency graph of projects within our workspace.
T> 
T> We can ask Nx to generate the dependency graph automatically, as well as highlight the parts of the graph that are affected by a given changeset.
T>
T> Nx can retest (ESLint, Jest, Cypress) and rebuild only the affected projects within our workspace.
T>
T> Nx automatically formats our code for us in an opinionated way using Prettier.
  
