# Husky, Eslint, Prettier pre-commit hook setup (with GitHub Desktop workaround)

Have you ever started a new JavaScript / Typescript project, opened up your editor and been overwhelmed with a rainbow of squiggly lines under missing imports, unformatted and unreadable code, `var`s instead of `let` or `const` everywhere? Have you tried preaching the benefits of Eslint and Prettier seemingly to closed ears?

I've been here many times, and sometimes you just have to fight back. My weapon of choice, the _pre-commit hook_! Wait until your nemesis tries to push a 2000 line React component with nothing but inline styles and TADA! eslint says NO! (Muahahaha!!...) Alright now that the venting and the evil laugh are out of the way, let's dive in. This blog post is for anyone that wants to setup a pre-commit hook that runs Eslint and Prettier on **staged changes** (aka when you try to commit to your branch). I also included some common configurations, and a workaround for users of GitHub Desktop and potentially other Git GUIs if they have the same issue.

Goodbye code smell!

## Install dependencies

Real quick, let's install some of the required dependencies for our hook to work as intended. Notice how we don't install Husky here. That's on purpose, we will install it later using `npx`.

```bsh
npm install --save-dev pretty-quick lint-staged
```

## Eslint

For our intents and purposes, we are going to use [lint-staged](https://github.com/okonet/lint-staged) to run Eslint on our staged changes. As you can see by the library name, it does exactly what we want, run eslint on `staged` changes only.

I won't go into detail on creating an eslint configuration file (`eslintrc.js` or such) as that is not a part of this tutorial and up to your individual project. We do however need a new configuration file for `lint-staged` specifically. This config file tells `lint-staged` what files to look for to run on and with which variation of the eslint command to use. let's create a `.lintstagedrc.json` file at the root of our repo.

Here we say "run eslint on all `.js`, `.jsx`, `.ts`, and `.tsx` files found from the root down". We also pass the --fix flag to eslint to allow it to automatically fix "fixable" eslint issues.

```json
{
  "./**/*.{js,jsx,ts,tsx}": "eslint --fix"
}
```

Passing `--fix` is optional. We can simply not pass the flag and make developers fix each error individually. In my opinion though, it is a better experience to let the linter fix what it can on it's own and then leave the rest to the developer.

If your files are located in a `src` folder and you want to ignore anything outside of that (like `build` or `dist` folders):

```json
{
  "{src}/**/*.{js,jsx,ts,tsx}": "eslint --fix"
}
```

Or a `server` and `client` folder:

```json
{
  "{server,client}/**/*.{js,jsx,ts,tsx}": "eslint --fix"
}
```

Easy enough, now on to Prettier.

## Prettier

We are going to use [pretty-quick](https://github.com/azz/pretty-quick) to run Prettier on our staged changes. Like with Eslint, this tutorial doesn't cover setting up a prettier config. Your preferred configurations are up to your individual project. However, one recommendation would be, don't force your style onto everyone in this case. Unlike with Eslint which has a functional purpose, prettier is just your code being vain and this is where other developers might have preferences that are worth agreeing upon.

## One more thing before setting up the hook

Now most advanced projects that don't use Prettier and Eslint until you show up might need a good dose of spring cleaning... so slap these next commands into your console and open up a "Spring Cleaning" PR to the horror of everyone else on the project complaining that you are changing every line of their terribly formatted and non standard code.

```bsh
// TODO

```

## Install Husky

Husky seems to be the defacto pre-commit hook library out there. I didn't do much research here and it works pretty well so I went with it. Let's first initialize our project with Husky:

```bsh
npx husky-init && npm install
```

Update .husky/pre-commit file to:

```bsh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

cd ./web
npx pretty-quick --staged
npx lint-staged
cd ./client
npx lint-staged
```

## GitHub Desktop error resolution:

```
TODO find error
```

We can solve this will a simple script. Create a `pre-commit-setup.js` file:

```js
const fs = require("fs");
const homedir = require("os").homedir();

fs.writeFileSync(homedir + "/.huskyrc", 'PATH="/usr/local/bin:$PATH"', {
  encoding: "utf-8",
});
```

and then in `package.json` add a new script. This script runs on postinstall so that after you run `npm install` this script will be executed, thus making sure that everyone has their env up to date.

```json
"postinstall": "node ./scripts/pre-commit-setup/index.js"
```

## TODO `--no-verify`

## BONUS: remove unused imports

To remove unused imports:
npm install eslint-plugin-unused-imports --save-dev
follow all of Usage: https://www.npmjs.com/package/eslint-plugin-unused-imports
