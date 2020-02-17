# Digital Product Design Development Team Documentation

## Table of Contents

- [Git Process](#git-process)
- [Using NPM within the network](#using-npm)
- [Transpilation Process](#transpilation-process)
- [Using ESLint](#linting-standards)
- [Prettier](#prettier)
- [BEM](#bem)
- [Performant Animation](#performant-animation)
- [NPM Scripts Boilerplate Toolchain](#boilerplate-npm-scripts-toolchain)
- [Audits in Chrome Dev Tools](#audits-in-devtools)
- [Using Gatsby Within WFA](#setting-up-a-new-gatsby-project-within-wfa)
- [Useful VS Code Plugins](#useful-vs-code-plugins)
- [Performant Font Loading Techniques](#performant-font-loading)
- [Prototyping in CodePen](#prototyping-in-codepen)

## Git process

Within the Digital Product Design development team, we use a Git Workflow commonly referred to as a "feature workflow". A detailed tutorial on the topic can be found [here](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow). Essentially, the theory behind the "feature workflow" is that all updates to the codebase take place in a feature branch and never on `master`. This assures that all updates do not conflict with `master` and that `master` never contains any broken code. The process for the workflow is described below.

### Start with the `master` branch

```
git checkout master
git fetch origin
git reset --hard origin/master
```

This switches to `master` and pulls the latest commits and resets the repo's local copy of `master` to match the lastest version.

### Create a new branch

```
git checkout -b new-feature
```

This switches to a branch called `new-feature` and creates one if it doesn't already exist. Feature branch names should be descriptive of the change that's being added. So, something like `redesign-accordion` or something else appropriate.

Make whatever changes or updates that are within the scope of the feature and make as many commits as are appropriate to adequately track the progress of your work. Make sure to regularly push your commits up to the remote repository to gather feedback from teammates and to provide a backup for your local copy.

```SHELL
git push -u origin new-feature
```

This will push the new feature branch to the `origin` remote and the `-u` flag adds it as a remote tracking branch. After the branch is set up as a tracking brach, `git push` can be run without any parameters to automatically push to the central repository.

### Get Feedback from the team / submit a pull request

Every pull request should have a reviewso your teammates can comment and approve the pushed commits. Resolve their comments locally, commit, and push the suggested changes to Github. Your updates appear in the pull request. Always add a reviewer and request feedback.

Once the comments and feedback from the team has been resolved, merge your branch into master using the following process:

1. Resolve any merge conflicts that surface
1. Make sure that your local `master` branch is up to date.
1. Merge your local feature branch into your local `master`.
1. Push the updated `master` to the remote repository.

## Using NPM

By default, NPM writes to your system folder, which require admin rights and the `sudo` command in your terminal. In order to use NPM without `sudo`, run the following command in your terminal:

```
mkdir ~/.npm-packages
npm config set prefix ~/.npm-packages
```

Then, check the configuration:

```
# this should show the versions
node -v
npm -v
# this should show npm and ng with no errors
npm list -g --depth=0
```

Once you have this set up, you will need to configure your `.npmrc` file to pull packages from Artifactory, the in-network NPM registry clone. Navigate to your root directory and open your `.npmrc` file in your text editor. You may need to type <kbd>⌘</kbd><kbd>↑</kbd><kbd>.</kbd> to show the hidden dot files in your finder. Once you have the `.npmrc` file open in your text editor, paste in the following code and save the file:

```
registry=http://dctartifactory.wellsfargo.com/artifactory/api/npm/npm-virtual/
phantomjs_cdnurl=http://dctartifactory.wellsfargo.com/artifactory/thirdparty-local/com/phantomjs
strict-ssl=false
shrinkwrap=false
```

You should now be able to use NPM to install packages.

## Transpilation process

Our NPM workflow is using [Parcel](https://parceljs.org/) to bundle ES6 modules and transpile them to ES5. To get started with Parcel, run this command in your terminal, which will make Parcel available globally on your machine.

```
npm install -g parcel-bundler
```

Next, you'll need to clone [this repository](https://github.wellsfargo.com/app-WFA-Public-Site-Development/NPM_Scripts_boilerplate/) and make sure to keep the folder structure consistent as each `watch` task looks in specific folders for the assets it will process. Once the repo is cloned, navigate to the folder from within your terminal and run `npm install`. This will install all the neccessary packages and make the included commands available.

In addition to the `npm run bundle` script which bundles and transpiles your JS, there is also a `npm run babel` command which watches a single `app.js` file and transpiles it. Parcel uses Babel for transpiliation under the hood, so there's little functional difference between these two tasks, but the option is there. Both Parcel and Babel rely on a Babel configuration that can, optionally, be a separate `.babelrc` file or, in our case, directly in the `package.json` file. This configuration specifies the plugins and presets that govern how Babel will process the files. Keeping the configuration in the `package.json` file eliminates the need for another configuration file. Full documentation for how to configure Babel can be found [here](https://babeljs.io/docs/en/configuration), but our basic implementation is setting the browser support.

## Linting standards

[ESLint](https://eslint.org/) is a highly configurable linting system that helps to enforce stylistic and technical consistency within your JavaScript and JSX files. It can be set up to match the coding standards of the team, but a good starting place is the [Air BNB Configuration](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb).

### Global configuration

To set up a clobal configuration, and run the following command:

```
npm install -g eslint-config-airbnb eslint-plugin-import eslint-plugin-react eslint-plugin-jsx-a11y eslint
```

> If you already have eslint installed, you can delete the final `eslint` from this command.
> The documentation that Air BNB provides list a command using `peerDependencies`, which doesn't seem to work within WFA.

The above command installs all of the required dependecies for the Air BNB configuration globally, in your root directory. You will then need to create an `.eslintrc` file which will establish your eslint rules. In your terminal, create the file from within your root directory.

```
cd ~/
touch .eslintrc
```

Once you've made your `.eslintrc` file, you'll need to open it in your text editor and set up the configuration. You may need to type <kbd>⌘</kbd><kbd>↑</kbd><kbd>.</kbd> to show the hidden dot files in your finder. Once you have the file open, paste the following code into it and save it.

```javascript
{
  "env": {
    "es6": true,
    "browser": true,
    "jquery": true
  },
  "extends": "airbnb",
  "rules": {
    "no-console": 0,
    "no-unused-vars": 1
  },
}
```

Once the packages are installed and you have a `.eslintrc` file, linting rules will apply to all projects.

### Per project configuration

The Air BNB configuration is a robust and strict setup that enforces many of the standards around ES2015+ and as such, may be a bit stringent for legacy files such as `common.js` and `script.js`. Luckily, eslint can be configured on a project by project basis. Project level configuration will override the global config. To use eslint locally, the project first needs to be set up with NPM. Run the following command in your terminal from within your project folder.

```
npm init
```

Fill out the fields as they are asked. If you'd rather have your `package.json` file pre-filled, run `npm init -y`. Once NPM is set up, run

```
npm install eslint-config-airbnb eslint-plugin-import eslint-plugin-react eslint-plugin-jsx-a11y eslint --save-dev
```

to install eslint locally in the project. Next, you'll need to initiate eslint from within the project.

```
npx eslint --init
```

> The npx
> You will be presented with five questions that will determine the initial set up of the eslint configuration. Once those have been answered, you will have an `.eslintrc.json` (if you chose JSON as the format for the file) in your project that will govern the linting rules. If you selected JSON as the format,it's initial state will look something like this:

```json
{
  "env": {
    "es6": true,
    "browser": true,
    "jquery": true
  },
  "extends": "airbnb",
  "rules": {
    "no-console": 0,
    "no-unused-vars": 1
  },
  "plugins": ["html", "markdown"]
}
```

Alternatively, you can choose to have your config file formatted in either JavaScript or YAML. The file format doesn't affect the configuration.

From here, you can configure the file in whatever manner is appropriate for the project. For more detail on configuring ESLint, see [this documentation](https://eslint.org/docs/user-guide/configuring).

### Ignoring certain files

Since eslint analyzes every JavaScript file in your project, you may want to avoid linting your transpiled code to avoid issues when it's time to commit changes. To facilitate this, eslint supports an `.eslintignore` file where you can specifiy files for eslint to ignore, similar to the `.gitignore` file. To enable this, add an `.eslintignore` file at the root of your project and specify which files you want to ignore. The `.eslintignore` file will take any file path pattern and can take a glob pattern to indicate a group of files. Here is an example:

```
src/js/compiled/*
```

This pattern will ignore any file in the compiled folder.

### Automatically fix issues

Make sure you have the [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) plugin installed in VS Code and paste the following into your `settings.json`:

```javascript
"eslint.autoFixOnSave": true,
  "eslint.validate": [
    {
      "language": "html",
      "autoFix": true
    },
    {
      "language": "javascript",
      "autoFix": true
    }
  ],
```

This will enable ESLint to automatically fix whatever it can. From there, you can go through your JS file and manually fix whatever issues remain.

### Prevent eslint failing code from Git commits

Each Git repository has a `.git` folder in it which provides the ability to leverage Git to enforce certain rules around commits. A particularly useful hook to use is one that does not allow any code that breaks eslint's rules to be committed. To enable this, first you will have to change your settings in VS Code to show the .git folder in your side bar. Add this to your `settings.json` file in VS Code:

```javascript
  "files.exclude": {
    "**/.git": false
  },
```

Once the .git folder is accessible, navigate to the hooks folder and change the file called `commit-msg.sample` to `commit-msg` and paste the following into the file:

```bash
#!/bin/bash
files=$(git diff --cached --name-only | grep '\.jsx\?$')

# Prevent ESLint help message if no files matched
if [[ $files = "" ]] ; then
exit 0
fi

failed=0
for file in ${files}; do
git show :$file | eslint $file
if [[ $? != 0 ]] ; then
  failed=1
fi
done;

if [[ $failed != 0 ]] ; then
echo "🚫🚫🚫 ESLint failed, git commit denied!"
exit $failed
fi
```

Now, when you try to commit any files that don't pass your eslint rules, an error will be displayed in your terminal until you fix the errors. This process assures that all code committed to a repo with the hook enabled will adhere to the agreed upon standards set by the team

## Prettier

Prettier is an automatic code formatting tool that ensures consistency in code style across the team. To use it, install the [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) VS Code plugin and paste the following code block into your `settings.json`:

```javascript
"editor.formatOnSave": true
"prettier.eslintIntegration": true,
"prettier.singleQuote": true,
"prettier.printWidth": 120,
```

Using Prettier and turning `editor.formatOnSave : true` will automatically apply the Prettier setting each time you save. This will reduce the size of diffs in your Git commits when working on projects with the team.

## BEM

In the Digital Product Design development team, we aim to adhere to the [BEM](http://getbem.com/) naming convention for our CSS selectors. BEM stands for Block Element Modifier and provides for a flexible, scalable CSS codebase without specificity conflicts. There have been times when we have to deviate from this convention, when modifying existing Tridion components for example, but such deviations should occur as rarely as possible in order to keep the CSS maintainable. The specific structure of a BEM class is block\_\_element--modifer. The element portion of the classname is separated from the block by two underscores and the modifer is separated from the element with two dashes. It is not neccessary to use every portion of the class naming convention for BEM to be effective.

The specific usage of BEM is slightly open to interpretion, but generally adheres to the following pattern. For example, in a `<nav>`, you might have the following stucture:

```HTML
<nav class="nav">
  <ul class="nav__list">
    <li class="nav__item">
      <a class="nav__link">Link one</a>
    </li>
      <a class="nav__link">Link two</a>
    </li>
      <a class="nav__link nav__link--highlight">Link three</a>
    </li>
  </ul>
</nav>
```

In the above example, the `<nav>` is the _Block_ and the `<ul>`, `<li>` and `<a>` tags represent the _Elements_. Note the additional class on the final `<a>` tag. It contains an addition on the word `highlight`, which represents the _Modifier_. This naming convention allows the CSS to use single classnames as the selectors, keeping specificity consistently flat.

```CSS
.nav {
  width: 100%;
}
.nav__list {
  display: flex;
  justify-content: space-evenly;
  list-style: none;
}
.nav__link {
  color: #00698c;
  padding: 1rem;
}
.nav__link--highlight {
  color: #bada55;
}
```

Here, the `.nav__link` _Elements_ will all have the padding, but the final link will have a different text color thanks to the _Modifier_. Another way to think about BEM is in relation to block elements. It wouldn't make much sense to have a specific class for every heading thoughout your HTML, so in some cases the element can be treated as the block.

```HTML
<h1 class="heading">This is a normal heading</h1>
<h1 class="heading heading--em">This is a modified heading</h1>
```

In the above example, each heading has a classname of `heading` to indicate the _Block_ and the second heading has an additional class of `heading--em` with the `--em` indicating a _Modifier_. The CSS would then be something like this:

```CSS
.heading {
  font-weight: normal;
  font-family: 'WF Serif', Georgia, serif;
}
.heading--em {
  text-style: italic;
}
```

Again, both headings would retain the same `font-weight` and `font-family`, but the heading with the `heading--em` class would be _modified_ to be in italics.

## Performant animation

Animating CSS properties can provide a significant drain on CPU due to the fact the most properties' animations take place on the main thread in the browser. There are currently two exceptions to this, the `transform` and `opacity` properties. Animating both of this properties take place off the main thread and therefor typically run at 60FPS. Whenever possible, only animate these two properties and never animate position based properties like `top`, `left`, `rigth`, `bottom`, `margin`, or `max-height`. Additionally, adding the `will-change` property to an element you plan on animating also allows the brower to prepare for the animation. More information about perfomant animations can be found [here](https://developers.google.com/web/fundamentals/design-and-ux/animations/animations-and-performance) and [here](https://developers.google.com/web/fundamentals/design-and-ux/animations/).

## Audits in devtools

Chrome developer tools have incorporated a tool known as 'Lighthouse' in the audits tab. To use it, click the audits tab and select the options you want to audit. Auditing performance and accessibility is a quick way to identify major performance bottlenecks and accessibility issues before involving QA. Pay particular attention to image size and render blocking JS and CSS as these are areas for easy performance optimization. Address any accessibility issues that Lighthouse flags before involving one of the WFA digital accessibility team embers.

## Boilerplate NPM Scripts toolchain

The development team has a starter setup of an [NPM script toolchain](https://github.wellsfargo.com/app-WFA-Public-Site-Development/NPM_Scripts_boilerplate/) that handles standard development tasks like scss compilation, running a development server and transpiling ES6+ to ES5. Below is the README for that toolchain which details how to get started with it in your project. Once you clone the repo, you'll want to change the name of the folder to reflect your project and then you'll need to update the remote repository.

Once the folder is renamed, go to the team Github account and create a new repository for your project. Then, from within the project folder in your terminal, run the following command.

```shell
git remote rm origin
```

Verify that the remote has been removed

```shell
git remote -v
```

This should show no existing remotes.

Then, add your newly created repository as your new `origin` remote.

```shell
git remote add origin [add the path to your new repo here]
```

## Usage

To get started, open your terminal and type <kbd>⌘</kbd><kbd>↑</kbd><kbd>.</kbd>. This key command shows all the hidden files within your macOS system.

1.  Navigate to your user folder in the finder and replace the contents of your `.npmrc` file with the block below

```
registry=https://dctartifactory.wellsfargo.com/artifactory/api/npm/npm-virtual/
phantomjs_cdnurl=http://dctartifactory.wellsfargo.com/artifactory/thirdparty-local/com/phantomjs
strict-ssl=false
shrinkwrap=false
```

> The step above is a global setting that you will need for any project utilizing NPM. It replaces the default NPM registry with the Wells Fargo Artifactory so all your NPM modules will come from within the network.

> Once the above step has been done once, it will not need to be repeated in future projects.

2.  Clone this repo, rename the folder to your project name and run `npm install` from within the project folder in your terminal.

### Available Scripts

This boilerplate includes a number of pre-configured NPM scripts for processing and building files. Examples:

`npm run watch` will start a local server in your default browser and load an index.htm file from the src directory (if one exists). It will then watch your scss and js files for changes. When an file in src/scss changes, it will be processed and output to the src/css directory. When a file in src/js changes, it will be run through babel and output into the src/js/compiled directory. The browser will reload automatically on save. The URL in your default browser can be open across all your browsers and it will reload in all of them simultaneously.

If you don't need a server and only want to process scss, run `npm run watch:css` from the project folder in your terminal. The task will watch for changes in your src/scss files and process them on save.

If you only want to transpile ES6+, run `npm run bundle` from the project folder and it will watch for changes in your src/js folder and process them on save.

If you only need a server, run `npm run serve` and it will open your index.html file in your default browser.

`npm run build` will minify you transpiled JS, autoprefix your compiled CSS, and generate source maps. It will also optimize any SVGs within your project. Finally, it will move all your files and assets from the src directory to the dist directory, where the files will be ready to deploy.

## Setting up a new Gatsby project within WFA

> This a copy of the documentation on the Gatsby site, so the tone is a bit different.

There are many Enterprise level companies that maintain an internal clone of the NPM registry for security purposes. If you work for such a company, you may find that you are able to successfully run `npm install -g gatsby-cli` but cannot run the `gatsby new <project-source>` as the `gatsby new` command clones a repo from a public GitHub repository. Many companies block public GitHub, which will cause the `gatsby new` command to fail. Not to worry, though, you can set up a new Gatsby site without the `gatsby new` command with a few quick steps.

### Preparing your environment

To get started with Gatsby, you’ll need to make sure you have the following software tools installed:

1. [Node.js](https://www.gatsbyjs.org/tutorial/part-zero/#install-nodejs)
1. [npm CLI](https://www.gatsbyjs.org/tutorial/part-zero/#familiarize-with-npm)
1. [Gatsby CLI](https://www.gatsbyjs.org/tutorial/part-zero/#install-the-gatsby-cli)

For step-by-step installation instructions and detailed explanations of the required software, head on over to the [Gatsby tutorial](https://www.gatsbyjs.org/tutorial/part-zero/).

After your developer environment is set up, you'll want to set up a new project folder.

```shell
mkdir my-new-gatsby-site
cd my-new-gatsby-site
```

Next, you'll need to set up NPM within your project.

```shell
npm init
```

Fill out the prompts for the `package.json` file that is generated. If you'd like to skip that, you can run `npm init -y` and a pre-filled `package.json` will be generated for you.

Now, you'll need to install the necessary packages that Gatsby relies on to work its magic.

```shell
npm install --save gatsby react react-dom
```

Next, you'll add a `src` directory and a `pages` directory inside your project.

```shell
mkdir src
cd src
mkdir pages
```

Inside the pages directory, you'll make an `index.js` file that exports a React component.

```shell
cd pages
touch index.js
```

Now, add some React code to your `index.js` file as a starting point for your project.

```jsx:title=src/pages/index.js
import React from 'react';

export default () => <h1>Hello Gatsby!</h1>;
```

Finally, go back to the root of your project and run the `gatsby develop` command to start a development server and begin coding.

```shell
cd ../../
gatsby develop
```

And that's it! You should now have your initial page running on `localhost:8000` with a GraphiQL IDE running on `localhost:8000/___graphql`. From here, you can follow the rest of the [Gatsby tutorial](https://www.gatsbyjs.org/tutorial/part-zero/#set-up-a-code-editor) starting with setting up a code editor to get the full experience of what Gatsby can offer.

## Useful VS code plugins

- [Auto Rename Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag) : Automatically renames the closing tag in either HTML or JSX as you rename the opening tag.
- [Better Comments](https://marketplace.visualstudio.com/items?itemName=aaron-bond.better-comments): Highlights comments in your JS based on key words like TODO
- [Bracket Pair Colorizer](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer): Makes finding the matching bracket, curly brace or angle bracket much more simple.
- [Color Highlight](https://marketplace.visualstudio.com/items?itemName=naumovs.color-highlight): adds highlighting to color values declared in your code. Also shows the colors in the sidebar of the editor's minimap.
- [ES7 React/Redux/GraphQL/React-native Snippets](https://marketplace.visualstudio.com/items?itemName=dsznajder.es7-react-js-snippets): Adds keyboard shortcuts for creating JSX components and ES2015+ code.
- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint): Integrates with your ESLint configuration to display errors and warnings in your JS files within VS Code.
- [Git Blame](https://marketplace.visualstudio.com/items?itemName=waderyan.gitblame): Shows which contributor on a given project was responsible for each line of code.
- [Git Lens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens): Adds additional Git functionality into VS Code.
- [htmltagwrap](https://marketplace.visualstudio.com/items?itemName=bradgashler.htmltagwrap): Allows you to wrap a line of text in HTML, type `option + W` and wrap the selection in HTML tags.
- [Import Cost](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost): Displays the file size of importing a module into your project. With this information, you can choose to import a specific method instead of a whole module if that's all you need.
- [npm Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.npm-intellisense): Provides suggestions as you type file paths for node_modules to more easily find and import to a given file.
- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode): Automatically formats files based on shared configuration.
- [SCSS Intellisense](https://marketplace.visualstudio.com/items?itemName=mrmlnc.vscode-scss): Advanced autocompletion for SCSS variables and partials.
- [SVG Viewer](https://marketplace.visualstudio.com/items?itemName=cssho.vscode-svgviewer): Provides a visual preview of SVG files within VS Code.
- [Toggle Quotes](https://marketplace.visualstudio.com/items?itemName=BriteSnow.vscode-toggle-quotes): Allows you to quickly toggle a string between single quotes, double quotes and backticks.
- [vscode-styled-components](https://marketplace.visualstudio.com/items?itemName=jpoissonnier.vscode-styled-components): Provides the same CSS syntax highlighting of a given VS Code theme within styled components.

## Performant Font Loading

When a browser encounters a font in a `font-family` property that matches a font declared in an `@font-face` block, the default behavior is to hide the text until the font loads and is parsed. Most browsers will wait for a web font for up to three seconds before displaying a fallback font. Older -webkit- browsers, however, have an unlimited render blocking period. So, for users on those browsers, a failure in web font loading means that users will not get any content. To mitigate that risk, there are a number of browser APIs to leverage for a more robust user experience.

### The `font-display` descriptor

The `font-display` descriptor can be included with your `@font-face` blocks and serves as an indicator to the browser about how to handle the period of time it takes for the web fonts to load. It takes five potential values.

1. `auto` tells the browser to use it's default behavior, which, as stated above includes a timeout period of invisible text. There is little benefit in using `font-display` if you are going to use this value.

1. `swap` immediately renders the fallback font and displays the web font whenever it loads, whether that is 300ms or 10s.

1. `block` blocks rendering of text for as long as the web font takes to load. This value is useful if the custom typeface is of utmost importance to the design.

1. `fallback` Acts as a compromise between the auto and swap values. The browser will hide the text for about 100ms and, if the font has not yet been downloaded, will use the fallback text. It will swap to the new font after it is downloaded, but only if the web font loads in under 3s.

1. `optional` will display the fallback font immediately and then will only display the web font if it loads in under 100ms. If the web font doesn't load in that window, it will still download in the background and can be cached for subsequent vists, either in browser cache or in a service worker.

### `<link rel="preload">`

The `rel="preload"` attribute allows you to tell the browser which assets of top priority to the experience so it can load those assets earlier in the rendering process. This can be used to load font files and CSS files associated with the web fonts. This attribute typically improves loading so much that the `font-display` descriptor never even gets used.

`<link rel="preload" href="fonts/path-to-font.woff2" as="font" type="font/woff2" crossorigin>`

`<link rel="preload" href="fonts.css" as="style">`

> Note that when using `rel="preload"` for fonts, you MUST use the `crossorigin` attribute for it to work properly.

> Also of note- be selective about what resources you preload. If everything is given priority, then nothing is prioritized.

### The CSS Font Loading API

The Font Loading API is a way to promagramatically add fonts to an HTML file so that they load asynchronously and do not block rendering. This is useful as a way to group font loading to reduce repaints, which can be jarring to a user. To use this API, call a new instance of the `FontFace` objeect and pass in three arguments, the name of the font, the path to the font file and an object containing all the other information included with an `@font-face` block. Here is an example:

```JavaScript
var font = new FontFace("WF Serif", "url(/fonts/wf-serif.woff2)", {
  style: 'normal',  weight: '400'
});

// don't wait for the render tree, initiate an immediate fetch!
font.load().then(function() {
  // apply the font (which may re-render text and cause a page reflow)
  // after the font has finished downloading
  document.fonts.add(font);
  document.body.style.fontFamily = "WF Serif, serif";

  // OR... apply your own render strategy here...
});
```

### Font Face observer

Font Face Observer is a polyfil for the Font Loading API and can be leveraged for browsers that do not support font loading. It's API aims to mimic the Font Loading API behavior but is different in one crucial respect. Font Face Observer does not load fonts asynchronously, but it does return a promise. Once the promise resolves, you can add a class to the `<body>` tag that references the web fonts in your CSS. This will allow you to group repaints and prevent your web fonts from blocking render. To use Font Face Observer, include the script with your project. It can found [here](https://github.com/bramstein/fontfaceobserver) along with full documentation for its usage. Then, add your call to `FontFaceObserver` in your script file or in the `<head>` of your HTML files. Below is an example of loading multiple fonts with Font Face Observer.

```JavaScript
var fontA = new FontFaceObserver('WF Sans');
var fontB = new FontFaceObserver('WF Serif');

Promise.all([fontA.load(), fontB.load()]).then(function () {
  document.body.classList.add('fonts-loaded');
});
```

And then in your CSS:

```CSS
.fonts-loaded {
  font-family: 'WF Sans', sans-serif;
}

.fonts-loaded h1 {
  font-family: 'WF Serif', Georgia, serif;
}
```

## Prototyping in CodePen

The Digital Product Design Development Team has a shared [CodePen](https://codepen.io) that can be used for rapid prototyping and sharing proofs of concept around the overall Digital Product Development group. When using CodePen for this purpose, make sure that your user is set to the team account. Then, make sure that your pens or projects are saved as private. When sharing prototypes with stakeholders, change the view of the pen to Live View and share the URL in the browser address bar. For projects, click the button containing the square with the arrow in it at the top of the preview window and share the URL of the browser window that opens.
