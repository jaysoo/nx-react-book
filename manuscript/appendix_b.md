{#appendix-b}
# Appendix B: Using `npm` instead of `yarn`

We make light use of `yarn` so most of this book should remain the same if you follow along using `npm`.

There are four places where you will need to run different commands.

1. `yarn create ...` becomes `npm init ...`.
2. `yarn add ...` becomes `npm install --save ...`.
3. `yarn global add ...` becomes `npm install -g ...`.
4. `nx ... becomes `npm run nx ...`.

In [Chapter 1](#chapter-1) when you create the workspace, you should run `npm init nx-workspace`.

The examples in this book assume you have `nx` installed globally. So go ahead and run `npm install -g @nrwl/cli`.

