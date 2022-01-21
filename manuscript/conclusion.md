{#conclusion}

# Conclusion

You reached the end of this book! By now you should have a good overview of what a monorepo is about, the advantages you potentially gain by implementing one and how Nx can help you succeed with that.

We covered

- How to setup a new Nx workspace optimized for React development
- How an Nx workspace is organized in applications and libraries
- How to categorize libraries into feature, ui and data access libraries 
- How Nx comes with a built-in setup for ESLint, Prettier, Jest and Cypress
- How to use Nx generators to scaffold new libraries and components
- How to share code between a React frontend and Node backend application
- How to use Nx's dependency graph to visualize the workspace structure
- How Nx "affected" commands can help drastically reduce the amount of required computation and help with scaling of our repository

This should give you a good starting point. But there's even more to discover:

- Custom Workspace generators, executors and Nx Plugins[^workspacegenerators]
- Computation caching[^computationcaching]
- Distributed task execution[^DTE]
- Migrating to Nx from various other tooling such as Lerna, Yarn workspaces, CRA,...
- And much more!

To learn more about Nx's features please visit the documentation website at [nx.dev](https://nx.dev).

[^workspacegenerators]: https://nx.dev/generators/workspace-generators
[^computationcaching]: https://nx.dev/using-nx/caching
[^DTE]: https://nx.dev/using-nx/dte
