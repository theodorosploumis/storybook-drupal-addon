# Drupal Storybook Addon

A library for best-practice Drupal integration addons in Storybook:

- Easy-to-use Drupal integration
- Simple drop-down menu
- URL-linkable state for sharing

![Screenshot](./assets/screenshot.png)

## Configure your Drupal site

### 🌳 Install the Drupal module

Install and enable the Drupal module:

```console
composer require drupal/cl_server;
drush pm:enable --yes cl_server;
```

### 🌴 Add Storybook to your Drupal repo

From the root of your repo:

```console
yarn global add sb@latest;
sb init --builder webpack5 --type server
# If you have a reason to use Webpack4 use the following instead:
# sb init --type server
yarn add -D @lullabot/storybook-drupal-addon
```

### 🌵 Configure Storybook

First enable the addon. Add it to the `addons` in the `.storybook/main.js`.

Set the `staticDirs` path to your Drupal root (usually `web`). This allows us to enable local development for CSS/JS assets.

```javascript
// .storybook/main.js
module.exports = {
  // ...
  addons: [
    // ...
    '@lullabot/storybook-drupal-addon',
  ],
  staticDirs: ['../web'],
  // ...
};
```

Then, configure the `supportedDrupalThemes`, `localDev` and `drupalTheme` parameters in `.storybook/preview.js`.

`supportedDrupalThemes` is an object where the keys are the machine name of the Drupal themes and the values are the plain text name of that Drupal theme you want to use. This is what will appear in the dropdown in the toolbar.

`localDev` is a boolean (default `false`) that indicates where Storybook JS/CSS assets should point to local folders. This allows a themer to preview the Storybook components using local CSS/JS files. Notice that this option just **replaces CSS/JS paths** on the Storybook rendered page. Twig files still come from the `server.url` Drupal website!

```javascript
// .storybook/preview.js
const preview: Preview = {
  // ...
  globals: {
    // ...
    localDev: true,
    drupalTheme: 'umami',
    supportedDrupalThemes: {
      umami: {title: 'Umami'},
      bartik: {title: 'Bartik'},
      claro: {title: 'Claro'},
      seven: {title: 'Seven'},
    },
  },
  parameters: {
    server: {
      // Replace this with your Drupal site URL, or an environment variable.
      url: 'http://local.contrib.com',
    },
    // ...
  },
  // ...
};

export default preview;
```

## Start Storybook

Start the development server Storybook server. Locate the static path to the Drupal root:

```console
yarn storybook
```

## Tips for writing YML stories

- Anotated example with the different options for writing stories: https://gitlab.com/-/snippets/2556203
- Closed issue with some nifty tips: https://github.com/Lullabot/storybook-drupal-addon/issues/34

---

## Storybook addon authors

As an addon author, you can use this library by adding it as a dependency and adding the following to your `src/manager.ts` and `src/preview.ts` files:

*src/manager.ts*
```typescript
export * from '@lullabot/storybook-drupal-addon/manager';
```

*src/preview.ts*
```typescript
import type { Renderer, ProjectAnnotations } from '@storybook/types';
import drupalPreview from '@lullabot/storybook-drupal-addon/preview';
import { withYourDrupalDecorator } from './withYourDecorator';

// @ts-ignore
const drupalDecorators = drupalPreview?.decorators || [];

const preview: ProjectAnnotations<Renderer> = {
    ...drupalPreview,
    decorators: [...drupalDecorators, withYourI18nDecorator],
}

export default preview;
```

The currently selected drupal theme is available in the `drupalTheme` global, so you can access it in a decorator using the following snippet:

```typescript
import { MyProvider } from 'your-drupal-library';
import { useGlobals } from '@storybook/manager-api';

const myDecorator = (story, context) => {
  const [{drupalTheme}] = useGlobals();
  
  return <MyProvider drupalTheme={drupalTheme}>;
}
```
