# Complete Vue Final project setup.

All the steps to setup you complete Vue project in one place. The aim is to have full testing support, have it deploy on GitHub Pages, have functional linting, TypeScript, and be able to push it to your Turing project repository. I skipped adding Vitest to globals.

## 1. Things you should already have.

### VS Code Extensions. (By id)

- vscode-icons-team.vscode-icons
- dbaeumer.vscode-eslint
- Zignd.html-css-class-completion
- christian-kohler.npm-intellisense
- christian-kohler.path-intellisense
- ms-playwright.playwright
- esbenp.prettier-vscode
- yoavbls.pretty-ts-errors
- stylelint.vscode-stylelint
- shardulm94.trailing-spaces
- pmneo.tsimporter
- Vue.vscode-typescript-vue-plugin
- Vue.volar

## 2. Things you need to do.

1. Create a new public GitHub repository with your project name.
1. `git clone` the repository in your machine
1. Create a new Vue project with `npm create vite@latest` > `Vue` > `Customize with create-vue`
   - TypeScript: Yes
   - JSX: No
   - Vue Router: Yes
   - Pinia state management: Maybe
   - Vitest unit testing: Yes
   - End-to-End testing: Yes
   - ESLint: Yes
   - Prettier: Yes
1. Run `npm install`
1. Run `npm init stylelint`
1. Run `npm install @vue/eslint-config-airbnb postcss-html stylelint-config-standard-vue --save-dev`
1. Update your ESLint configuration: Add the '@vue/eslint-config-airbnb' package to the extends array in the `.eslintrc.cjs` file, some extra rules and @ path support. Example from 'TypeScript NASA APOD Hands-on'.

   ```cjs
   /* eslint-env node */
   require('@rushstack/eslint-patch/modern-module-resolution')

   const path = require('node:path')

   module.exports = {
     root: true,
     extends: [
       'plugin:vue/vue3-essential',
       'eslint:recommended',
       '@vue/eslint-config-airbnb',
       '@vue/eslint-config-typescript',
       '@vue/eslint-config-prettier/skip-formatting',
     ],
     parserOptions: {
       ecmaVersion: 'latest',
     },
     rules: {
       // TypeScript does a better job for import checks than ESLint
       'no-use-before-define': ['error', { functions: false }],
       'import/no-extraneous-dependencies': 'off',
       'import/extensions': 'off',
     },
     settings: {
       // to make our custom @ alias resolvable by ESLint import rules
       'import/resolver': {
         [require.resolve('eslint-import-resolver-node')]: {},
         [require.resolve('eslint-import-resolver-custom-alias')]: {
           alias: {
             '@': `${path.resolve(__dirname, './src')}`,
           },
           extensions: ['.mjs', '.js', '.jsx', '.json', '.node', '.ts', '.tsx'],
         },
       },
     },
   }
   ```

1. Update your Prettier configuration in `.prettierrc.json` according to your preference. My personal:
   ```json
   {
     "$schema": "https://json.schemastore.org/prettierrc",
     "semi": false,
     "tabWidth": 2,
     "singleQuote": true,
     "arrowParens": "avoid",
     "printWidth": 100,
     "trailingComma": "all"
   }
   ```
1. Update your stylelint configuration: The extends array in the `.stylelintrc.json` file should include `stylelint-config-standard-vue` and `stylelint-config-standard`.
   ```json
   {
     "extends": ["stylelint-config-standard", "stylelint-config-standard-vue"]
   }
   ```
1. Create project editor settings to auto lint on save. If you use auto save, I recommend to change VS Code save mode to `onFocusChange` so it does not change as you type. Create `.vscode/settings.json`
   ```json
   {
     "editor.codeActionsOnSave": {
       "source.fixAll": true,
       "source.fixAll.eslint": true,
       "source.fixAll.stylelint": true
     },
     "editor.formatOnSave": true,
     "eslint.validate": ["javascript", "javascriptreact", "typescript", "typescriptreact", "vue"],
     "stylelint.validate": ["css", "scss", "vue"]
   }
   ```
1. (Unclear) Update TypeScript config. While this is not stated anywhere explicitly, "Typed TypeScript NASA APOD Hands-on Solution" has more rules in `tsconfig.app.json`. Consider adding this as strict mode is generally recommended for TypeScript.
   ```json
   {
     "extends": "@vue/tsconfig/tsconfig.dom.json",
     "include": ["env.d.ts", "src/**/*", "src/**/*.vue"],
     "exclude": ["src/**/__tests__/*"],
     "compilerOptions": {
       "baseUrl": ".",
       "composite": true,
       "strict": true,
       "noUnusedLocals": true,
       "noUnusedParameters": true,
       "noImplicitReturns": true,
       "lib": ["ES2022", "DOM", "DOM.Iterable"],
       "paths": {
         "@/*": ["./src/*"]
       }
     }
   }
   ```
1. (Unclear) Minor change to `tsconfig.node.json`, adding strict mode.
   ```json
   {
     "extends": "@tsconfig/node18/tsconfig.json",
     "include": [
       "vite.config.*",
       "vitest.config.*",
       "cypress.config.*",
       "nightwatch.conf.*",
       "playwright.config.*"
     ],
     "compilerOptions": {
       "composite": true,
       "module": "ESNext",
       "moduleResolution": "Bundler",
       "types": ["node"],
       "strict": true
     }
   }
   ```
1. (Optional) Add style linting to your package scripts: Add `"lint:style": "npx stylelint \"**/*.(s?css|vue)\"",` to the scripts object in your package.json file. This will allow you to run `npm run lint:style` to lint all of your styles in CSS and Vue files.
1. (Unclear) Change the `type-check` in your package.json file ,to: `"type-check": "vue-tsc --noEmit -p tsconfig.app.json --composite false",`, otherwise it does not build.

   ```json
     {
       // ...
       "type-check": "vue-tsc --noEmit -p tsconfig.app.json --composite false",
       "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --fix --ignore-path .gitignore",
       "lint:style": "npx stylelint \"**/*.(s?css|vue)\"",
       "format": "prettier --write src/"
     },
   ```

1. (Unclear) `vitest.config.ts` now throws an error in imports. In Vitest exercise files it is just disabled, so here we do the same.

   ```javascript
   import { fileURLToPath } from 'node:url'
   /* eslint-disable import/no-unresolved */
   import { mergeConfig, defineConfig, configDefaults } from 'vitest/config'
   import viteConfig from './vite.config'

   export default mergeConfig(
     viteConfig,
     defineConfig({
       test: {
         environment: 'jsdom',
         exclude: [...configDefaults.exclude, 'e2e/*'],
         root: fileURLToPath(new URL('./', import.meta.url)),
       },
     }),
   )
   ```

1. Follow the [Volar Takeover Mode](https://vuejs.org/guide/typescript/overview.html#volar-takeover-mode) section in the Vue TypeScript guide. This will ensure Volar is used for all Vue TypeScript-related features instead of the built-in TypeScript language server.
1. Make sure you can run the project locally with npm run dev. Try to build the project. At this point you should be able to build and preview it without problems. Once it works, you can start setting up your GitHub Pages.
1. Go to your GitHub project's Settings tab. Select Pages in the sidebar. Under the Source section, which should be at the top, select "GitHub Actions". Also, open Actions in the Settings page sidebar and make sure you have "Allow all actions and reusable workflows" selected.
1. Add the GitHub Actions file .github/workflows/deploy.yaml using the template. Includes JavaScript Routes fix, ESlint linting, Vitests, E2E tests.

   ```yaml
   # Simple workflow for deploying static content to GitHub Pages
   name: Deploy static content to Pages

   on:
     # Runs on pushes targeting the default branch
     push:
       branches: ['main']

     # Allows you to run this workflow manually from the Actions tab
     workflow_dispatch:

   # Sets the GITHUB_TOKEN permissions to allow deployment to GitHub Pages
   permissions:
     contents: read
     pages: write
     id-token: write

   # Allow one concurrent deployment
   concurrency:
     group: 'pages'
     cancel-in-progress: true

   jobs:
     deploy:
       environment:
         name: github-pages
         url: ${{ steps.deployment.outputs.page_url }}
       runs-on: ubuntu-latest
       steps:
         ## Setup
         - name: Checkout
           uses: actions/checkout@v3
         - name: Set up Node
           uses: actions/setup-node@v3
           with:
             node-version: 18
             cache: 'npm'
         - name: Install dependencies
           run: npm install

         ## Lint
         - name: Run linters
           run: npm run lint

         ## Unit tests
         - name: Run unit tests
           run: npm run test:unit

         ## Build the source code
         - name: Build
           run: npm run build
         - name: Create 404 page
           run: cp ./dist/index.html ./dist/404.html
         - name: Upload artifact
           uses: actions/upload-pages-artifact@v1
           with:
             # Upload dist repository
             path: './dist'

         ## E2E test our built application
         - name: Install Playwright Browsers
           run: npx playwright install --with-deps
         - name: Run E2E tests
           run: npm run test:e2e
         - uses: actions/upload-artifact@v3
           if: always()
           with:
             name: playwright-report
             path: playwright-report/
             retention-days: 7

         ## Deploy
         - name: Setup Pages
           uses: actions/configure-pages@v3
         - name: Deploy to GitHub Pages
           id: deployment
           uses: actions/deploy-pages@v1
   ```

1. Edit `vite.config.ts` to add `GITHUB_REPOSITORY`.

   ```javascript
   import { fileURLToPath, URL } from 'node:url'

   import { defineConfig } from 'vite'
   import vue from '@vitejs/plugin-vue'

   // https://vitejs.dev/config/
   export default defineConfig({
     base: process.env.GITHUB_REPOSITORY ? `/${process.env.GITHUB_REPOSITORY.split('/')[1]}/` : '/',
     plugins: [vue()],
     resolve: {
       alias: {
         '@': fileURLToPath(new URL('./src', import.meta.url)),
       },
     },
   })
   ```

1. Edit `playwright.config.ts` to support `GITHUB_REPOSITORY`.

   - after our imports, let's add a small function that will dynamically form our baseURL

   ```typescript
   const formBaseURL = (baseURL: string) => {
     const repo = process.env.GITHUB_REPOSITORY
     const basePath = repo ? `/${repo.split('/')[1]}/` : ''

     return `${baseURL}${basePath}`
   }
   ```

   - find the line where baseURL is defined and replace it with

   ```typescript
   baseURL: formBaseURL('http://localhost:5173'),
   ```

   - (Optional) Set the timeout in playwright.config.ts to 2000 and expect timeout to 500 to speed up failing tests.

1. At this point, the standard Vue page should build on GitHub and be available online for everyone.

1. Clean up provided code. In `./src` you can ether delete `assets`, `components`, `views`, and clear outs `App.vue`, or you can adapt them to your needs. Remove `/about` from `routes`.

1. Let us also prepare our project to upload to Turing submissions while we're at it. Go to your designated repository and download your `.ipynb` assignment file. We will overwrite this repo later, and if it's not there, you will not be able to see your assignment on the platform. (Hopefully the way this work will change in the future.)

1. You are not allowed to upload your `.ipynb` to your public repo.
    > @Giedrius Zebrauskas on discord:
    > ... you potentially leak Turing tasks; show that this was not fully your own personal work but something for studies (you can of course mention that in interviews, but there's no need to make it obvious in the repo itself)

    So save it in the project folder and and add `*.ipynb` to your `.gitignore` file.

1. Copy Turing Submissions remote link, add it as a separate remote with git command

   ```
   git remote add REPONAME https://github.com/OWNER/REPOSITORY.git
   ```

   so it would be something like

   ```
   git remote add turing https://github.com/TuringCollegeSubmissions/<your-project-number>.git
   ```

   Now your public remote is called `origin` and is the default one, while `turing` is the private submission repo.

1. Since this is a different project from what turing created, we need to overwrite their repo.

   ```
   git push turing main -f
   ```

1. After pushing your `turing` repo, we will add our `.ipynb` in there manually through the Git website. For now, this is the simplest way.


1. We can now simply `git push origin main` to update our website, `git push turing main` to submit you project.


## (Optional) Create a git branch for your turing repository.

1. Create a new branch and move there

   ```
   git checkout -b for-turing
   ```

1. Place your `.ipynb` file into the directory or remove `*.ipynb` from `.gitignore`.

1. Remove or disable .github/workflows/deploy.yaml. It break in a privet repo that does not have Pages enabled.

1. Commit the changes. Now you should have a modified for-turing branch.

1. We will now make our this brach push into Turing repo by default, and also onto the main branch there so reviewers don't have to switch.

    ```
    git push --set-upstream turing for-turing:main
    ```

1. You can now run `git push`. It should update your turing repo's main branch.

1. Overview:
    - `git checkout main`: Update all your code here
    - `git push` from `main`: Update your public repo.
    - `git checkout for-turing`: Go to your turing branch
    - `git rebase main` from `for-turing`: Get all the latest updates from main
    - `git push` from `for-turing`: Update the private review repo.