---
title: Setting up a Yarn monorepo with Ignite
description: How to set up a Yarn monorepo using Ignite and two extra shared utilities
tags:
  - Ignite
  - Monorepo
  - Yarn
last_update:
  author: Felipe Peña
publish_date: 2024-08-22
---

# Setting up a Yarn monorepo with Ignite

👋 Hello and welcome to this monorepo guide! We know setting up a project using a monorepo structure can be sometimes challenging, therefore we created this guide to lead you through process. We'll be focusing on [React Native](https://reactnative.dev/) projects using the [Ignite](https://github.com/infinitered/ignite) framework and the [Yarn](https://yarnpkg.com) tool.

This guide starts by setting up the monorepo structure, then create a React Native app using the Ignite CLI, to finally end up generating two shared utilities: a form-validator utility and a shared UI library, that we will be integrate into the app.

## Prerequisites

Before we begin, we want to ensure you have these standard tools installed on your machine:

- [Node.js](https://nodejs.org/en) (version 18 or later)
- [Yarn](https://yarnpkg.com) (version 3.8 or later)

Now, let’s dive into the specific use case this guide will address.

## Use case

In a monorepo setup with multiple applications, like a React Native mobile app and a React web app, can share common functionalities.

In this guide we will be focusing on that premise and creating/utilizing shared utilities within the monorepo. For instance, if you have several apps that need to share an ESLint configuration or UI components, you can create reusable packages that can be integrated across all your apps.


:::info

Wait! How do I even know if my project will benefit from a monorepo structure? No worries! We have more documentation on monorepo tools and whether you want to choose this way of organization. You can find it [here](/docs/recipes/MonoreposOverview).

:::

By centralizing these utilities, you can reduce code duplication and simplify maintenance, ensuring that any updates or bug fixes are immediately available to all your apps.

In this setup, we’ll create a React Native app along with two shared packages: one for holding a common ESLint configuration and another for shared UI components. Finally, we’ll integrate these packages into the mobile app.

## Step 1: Setting up the monorepo

First, follow the [Expo documentation on setting up monorepos](https://docs.expo.dev/guides/monorepos/) to initialize your own monorepo. This will include setting up your `packages/` and `apps/` directories and configuring Yarn workspaces.

1. Initialize the monorepo:

```shell
mkdir monorepo-example
cd monorepo-example
yarn init -y
```

2. Configure workspaces in `package.json`:

```json
{
  "name": "monorepo-example",
  // error-line
  "packageManager": "yarn@3.8.4"
  // success-line-start
  "packageManager": "yarn@3.8.4",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ]
  // success-line-end
}
```

:::info

You can organize the folder structure of your Yarn monorepo however it best suits your project. While this guide suggests using `apps/` and `packages/`, you can rename or add directories like `services/` or `libs/` to fit your workflow.

The key is to keep your monorepo clear and organized, ensuring that it’s easy to manage and navigate for your team.

:::

3. Create directory structure:

```shell
mkdir apps packages
```

## Step 2: Create mobile app using Ignite

[Ignite](https://github.com/infinitered/ignite) is a battle-tested React Native boilerplate that Infinite Red uses every time we start a new project. In this step, we'll create a React Native app within the monorepo using Ignite's CLI.

1. Install the [Ignite CLI](https://www.npmjs.com/package/ignite-cli) (if you haven't already):

```shell
npx ignite-cli@latest
```

2. Generate a new app:
   Navigate to the apps/ directory and run the following command to create a new app:

```shell
cd apps
npx ignite-cli new mobile
```

We suggest the following answers to the prompts:

```
📝 Do you want to use Expo?: Expo - Recommended for almost all apps [Default]
📝 Which Expo workflow?: Expo Go - For simple apps that don't need custom native code [Default]
📝 Do you want to initialize a git repository?: No
📝 Remove demo code? We recommend leaving it in if it's your first time using Ignite: No
📝 Which package manager do you want to use?: yarn
📝 Do you want to install dependencies?: No
```

3. Open the `metro.config.js` file:

```shell
touch mobile/metro.config.js
```

4. Replace the following lines in the Metro configuration file with the lines below (this is taken from the [Expo guide](https://docs.expo.dev/guides/monorepos/)):

```js
// Learn more https://docs.expo.io/guides/customizing-metro
const { getDefaultConfig } = require('expo/metro-config');

// success-line-start
// Get monorepo root folder
const monorepoRoot = path.resolve(projectRoot, '../..');
// success-line-end

/** @type {import('expo/metro-config').MetroConfig} */
// error-line
const config = getDefaultConfig(__dirname);
// success-line
const config = getDefaultConfig(projectRoot);

config.transformer.getTransformOptions = async () => ({
  transform: {
    // Inline requires are very useful for deferring loading of large dependencies/components.
    // For example, we use it in app.tsx to conditionally load Reactotron.
    // However, this comes with some gotchas.
    // Read more here: https://reactnative.dev/docs/optimizing-javascript-loading
    // And here: https://github.com/expo/expo/issues/27279#issuecomment-1971610698
    inlineRequires: true,
  },
});

// success-line-start
// 1. Watch all files within the monorepo
config.watchFolders = [monorepoRoot];
// 2. Let Metro know where to resolve packages and in what order
config.resolver.nodeModulesPaths = [
  path.resolve(projectRoot, 'node_modules'),
  path.resolve(monorepoRoot, 'node_modules'),
];
// success-line-end

// This helps support certain popular third-party libraries
// such as Firebase that use the extension cjs.
config.resolver.sourceExts.push("cjs")

module.exports = config;
```

## Step 3: Install dependencies

Let's make sure all of our dependendencies are installed for the mobile app.

1. Run `yarn` at the root of the project:

```shell
cd ..
yarn install
```

## Step 4: Add a shared ESLint configuration with TypeScript

In our experience, maintaining consistent code quality across TypeScript projects within a monorepo is essential. Sharing a single ESLint configuration file between these apps ensures consistent coding standards and streamlines the development process. Let's go ahead and create an utility for that.

1. Create a shared ESLint configuration package:

Inside your monorepo, create a new package for your shared ESLint configuration.

```shell
mkdir packages/eslint-config
cd packages/eslint-config
```

2. Initialize the package:

Initialize the package with a `package.json` file.

```shell
yarn init -y
```

3. Install ESLint and TypeScript dependencies:

Install ESLint, TypeScript, and any shared plugins or configurations that you want to use across the apps. We recommend the follow:

```shell
yarn add eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-plugin-react eslint-plugin-react-native eslint-plugin-reactotron eslint-config-standard eslint-config-prettier --dev
```

4. Create the `tsconfig.json` file:

`packages/eslint-config/tsconfig.json`

```json
// success-line-start
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es6",
    "lib": ["es6", "dom"],
    "jsx": "react",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
 }
 // success-line-end
 ```

5. Create the shared ESLint configuration file:

Create an `index.ts` file in the root of your `eslint-config` package. We will reuse Ignite’s boilerplate ESLint configuration and then replace the original configuration with it.

`packages/eslint-config/index.ts`

```typescript
module.exports = {
  root: true,
  parser: "@typescript-eslint/parser",
  extends: [
    "plugin:@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-native/all",
    "standard",
    "prettier",
  ],
  plugins: [
    "@typescript-eslint",
    "react",
    "react-native",
    "reactotron",
  ],
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
  },
  settings: {
    react: {
      pragma: "React",
      version: "detect",
    },
  },
  globals: {
    __DEV__: false,
    jasmine: false,
    beforeAll: false,
    afterAll: false,
    beforeEach: false,
    afterEach: false,
    test: false,
    expect: false,
    describe: false,
    jest: false,
    it: false,
  },
  rules: {
    "@typescript-eslint/ban-ts-ignore": 0,
    "@typescript-eslint/ban-ts-comment": 0,
    "@typescript-eslint/explicit-function-return-type": 0,
    "@typescript-eslint/explicit-member-accessibility": 0,
    "@typescript-eslint/explicit-module-boundary-types": 0,
    "@typescript-eslint/indent": 0,
    "@typescript-eslint/member-delimiter-style": 0,
    "@typescript-eslint/no-empty-interface": 0,
    "@typescript-eslint/no-explicit-any": 0,
    "@typescript-eslint/no-object-literal-type-assertion": 0,
    "@typescript-eslint/no-var-requires": 0,
    "@typescript-eslint/no-unused-vars": [
      "error",
      {
        argsIgnorePattern: "^_",
        varsIgnorePattern: "^_",
      },
    ],
    "comma-dangle": 0,
    "multiline-ternary": 0,
    "no-undef": 0,
    "no-unused-vars": 0,
    "no-use-before-define": 0,
    "no-global-assign": 0,
    "quotes": 0,
    "react-native/no-raw-text": 0,
    "react/no-unescaped-entities": 0,
    "react/prop-types": 0,
    "space-before-function-paren": 0,
    "reactotron/no-tron-in-production": "error",
  },
}
// success-line-end
```

This configuration (originally sourced from [Ignite](https://github.com/infinitered/ignite)) provides a strong foundation for TypeScript, React and React Native projects. You can also adjust the rules according to your project's specific needs.

5. Compile the TypeScript configuration:

```shell
npx tsc
```

This will generate a `index.js` file from your `index.ts` file.

## Step 6: Use the shared ESLint configuration in the mobile app

1. Navigate to the mobile app:

```shell
cd ..
cd ..
cd apps/mobile
```

2. Add the ESLint shared package to the `package.json` file:

`apps/mobile/package.json`

```json
"eslint": "8.17.0",
// success-line
 "eslint-config": "workspace:^",
 "eslint-config-prettier": "8.5.0",
```

:::info

Although this guide focuses on private monorepos, it's important to address external publishing scenarios. For monorepos with some packages intended for public release, avoid using `workspace:^`. Instead, specify the exact version of each package to ensure proper consumption. To manage versioning and publishing of multiple packages within a monorepo, we recommend using the [changesets](https://github.com/changesets/changesets) tool.

:::

3. Replace the shared ESLint configuration in `package.json`:

`apps/mobile/package.json`

```json
// error-line-start
"eslintConfig": {
    "root": true,
    "parser": "@typescript-eslint/parser",
    "extends": [
      "plugin:@typescript-eslint/recommended",
      "plugin:react/recommended",
      "plugin:react-native/all",
      "standard",
      "prettier"
    ],
    "plugins": [
      "@typescript-eslint",
      "react",
      "react-native",
      "reactotron"
    ],
    "parserOptions": {
      "ecmaFeatures": {
        "jsx": true
      }
    },
    "settings": {
      "react": {
        "pragma": "React",
        "version": "detect"
      }
    },
    "globals": {
      "__DEV__": false,
      "jasmine": false,
      "beforeAll": false,
      "afterAll": false,
      "beforeEach": false,
      "afterEach": false,
      "test": false,
      "expect": false,
      "describe": false,
      "jest": false,
      "it": false
    },
    "rules": {
      "@typescript-eslint/ban-ts-ignore": 0,
      "@typescript-eslint/ban-ts-comment": 0,
      "@typescript-eslint/explicit-function-return-type": 0,
      "@typescript-eslint/explicit-member-accessibility": 0,
      "@typescript-eslint/explicit-module-boundary-types": 0,
      "@typescript-eslint/indent": 0,
      "@typescript-eslint/member-delimiter-style": 0,
      "@typescript-eslint/no-empty-interface": 0,
      "@typescript-eslint/no-explicit-any": 0,
      "@typescript-eslint/no-object-literal-type-assertion": 0,
      "@typescript-eslint/no-var-requires": 0,
      "@typescript-eslint/no-unused-vars": [
        "error",
        {
          "argsIgnorePattern": "^_",
          "varsIgnorePattern": "^_"
        }
      ],
      "comma-dangle": 0,
      "multiline-ternary": 0,
      "no-undef": 0,
      "no-unused-vars": 0,
      "no-use-before-define": 0,
      "no-global-assign": 0,
      "quotes": 0,
      "react-native/no-raw-text": 0,
      "react/no-unescaped-entities": 0,
      "react/prop-types": 0,
      "space-before-function-paren": 0,
      "reactotron/no-tron-in-production": "error"
    }
  }
// error-line-end
// success-line-start
"eslintConfig": {
  extends: ["@monorepo-example/eslint-config"],
}
// success-line-end
```

In this guide, we use `@monorepo-example` as the placeholder name for the monorepo. Be sure to replace it with your actual monorepo name if it’s different.

## Step 7: Create the shared UI components package

Let's create a Badge component as a shared UI component that can be used within the mobile app. For context, a Badge component is a simple and versatile element often used to display small bits of information, such as notifications, statuses, or labels.

1. Navigate to the packages folder:

```shell
cd ..
cd ..
cd packages
```

2. Create the package directory:

```shell
mkdir ui-components
cd ui-components
```

3. Initialize the package:

Initialize the package with a `package.json` file.

```shell
yarn init -y
```

4. Install dependencies:

Install any necessary dependencies, such as React, React Native, and TypeScript, which will be used across both platforms.

```shell
yarn add react react-native typescript --peer
yarn add @types/react @types/react-native --dev
```

4. Create the `tsconfig.json` file:

`packages/ui-components/tsconfig.json`

```json
// success-line-start
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "es2017"],
    "module": "commonjs",
    "jsx": "react",
    "declaration": true,
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
 // success-line-end
 ```

5.  Create the badge component:

Inside the `packages/ui-components` directory, create a `src` folder and add your Badge component.

```shell
mkdir src
touch src/Badge.tsx
```

6. Build the badge component:

`packages/ui-component/src/Badge.tsx`

```tsx
// success-line-start
import React, { FC } from "react"
import { View, Text, StyleSheet, ViewStyle, TextStyle } from "react-native"

interface BadgeProps {
  label: string
  color?: string
  backgroundColor?: string
  style?: ViewStyle
  textStyle?: TextStyle
}

export const Badge: FC<BadgeProps> = ({ label, color = "white", backgroundColor = "red", style, textStyle }) => {
  return (
    <View style={[styles.badge, { backgroundColor }, style]}>
      <Text style={[styles.text, { color }, textStyle]}>{label}</Text>
    </View>
  )
}

const styles = StyleSheet.create({
  badge: {
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 12,
    alignSelf: "flex-start",
  } satisfies ViewStyle,
  text: {
    fontSize: 12,
    fontWeight: "bold",
  } satisfies TextStyle,
})
// success-line-end
```

The way it's been defined above, a Badge component should be a simple UI element that can display a label with customizable colors, making it versatile for use in different parts of your application, such as notification counts, statuses, or category labels.

7. Export the badge component:

Ensure that your component is exported in the package's main entry file.

`packages/ui-component/src/index.ts`

```ts
// success-line-start
export * from "./Badge"
// success-line-end
```

8. Compile the package:

Compile your TypeScript code to ensure it's ready for consumption by other packages.

```shell
npx tsc
```

## Step 8: Use the shared UI package in the mobile app

1. Navigate now to the mobile app:

```shell
cd ..
cd ..
cd apps/mobile
```

2. Add the shared UI package to the `package.json` file:

`apps/mobile/package.json`

```json
    "react-native-screens": "3.31.1",
    // error-line
    "react-native-web": "~0.19.6"
    // success-line-start
    "react-native-web": "~0.19.6",
    "ui-components": "workspace:^"
    // success-line-end
  },
```

3. Add the Badge component to the UI

Let's now use the Badge component within the app. For this example, let's place it in the login screen, below the heading and above the form fields to indicate the number of login attempts if they exceed a certain number.

`apps/mobile/apps/screens/LoginScreen.tsx`

```tsx
import { AppStackScreenProps } from "../navigators"
import { colors, spacing } from "../theme"
// success-line
import { Badge } from "ui-components"

...

<Text testID="login-heading" tx="loginScreen.logIn" preset="heading" style={themed($logIn)} />
// success-line-start
{attemptsCount > 0 && (
  <Badge
    label={`Attempt ${attemptsCount}`}
    backgroundColor={attemptsCount > 2 ? "red" : "blue"}
  />
)}
// success-line-end
```

## Step 9: Run mobile app to make sure logic was added

1. Navigate to the root of the project:

```shell
cd ..
cd ..
```

2. Make sure dependencies are installed:

```shell
yarn
```

3. Run the React Native app (make sure you have your [environment setup](https://reactnative.dev/docs/set-up-your-environment)):

For iOS:

```shell
cd apps/mobile
yarn ios
```

For Android:

```shell
cd apps/mobile
yarn android
```

You should be able to view the login screen with an instance of a badge element added between the heading and the form fields.

## Step 10: Add Yarn global scripts (optional)

Yarn's workspaces feature allows you to define and run scripts globally across all packages in your monorepo. This simplifies your workflow by enabling you to execute tasks like testing, building, or linting from the root of your project, ensuring consistency across all packages. In this optional section, we’ll explore how to set up and use global scripts with Yarn in your monorepo.

Let's add a global script for the mobile app to run iOS and Android projects.


1. Add a global script to the mobile app `package.json` file:

`apps/mobile/package.json`

```json
  "scripts": {
    ...
    "serve:web": "npx server dist",
    // error-line
    "prebuild:clean": "npx expo prebuild --clean"
    // success-line-start
    "prebuild:clean": "npx expo prebuild --clean",
    "mobile:ios" : "yarn workspace mobile ios",
    "mobile:android" : "yarn workspace mobile android"
    // success-line-end
  },
```

Even though this script is locally defined within the app's `package.json` file, it will available everywhere within the monorepo by running `yarn mobile:ios` or `yarn mobile:android`.

For more information on Yarn's global scripts, check [this site](https://yarnpkg.com/features/workspaces#global-scripts).

## Conclusion

Congratulations on setting up your Yarn monorepo! By using the [Ignite](https://github.com/infinitered/ignite) framework, and two shared packages, you've successfully integrated these together. This setup enables you to scale your projects efficiently by sharing code across multiple applications in a well-structured and organized manner.

For more information, you can check the following resources:
* [Choosing the right monorepo strategy for your project](/docs/MonoreposOverview.md)
* [Expo: Work with monorepos](https://docs.expo.dev/guides/monorepos/)