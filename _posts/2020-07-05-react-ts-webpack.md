---
layout: post
title:  "Basic project setup for Typescript with React"
date:   2020-07-05 16:37:55 -0500
categories: typescript
---

I spent some time today setting up a project using [React](https://reactjs.org/), [Typescript](https://www.typescriptlang.org/), and [Webpack](http://webpack.github.io/).  There are various guides out there but none were quite to my liking.  It took a little while to get everything working right so I decided to document my journey.  Mostly I just did this for my own purposes but maybe it will be helpful to someone.  I'll be breaking it down step by step with every error along the way documented.  If you are looking for a quick get up and running guide and you don't care about any of the details, this is probably not the place for you.  If you are here to resolve some issue you have had or maybe want some insight into how things work and why they need to be configured the way they are, this post *might* help you.

Why did I do this?  For one thing, HtmlWebPackPlugin was not mentioned in many posts. I find that this plugin helps to cleanup the syntax of wiring everything together.  Typescript's own website [recommends linking React directly from node modules in your index file](https://www.typescriptlang.org/docs/handbook/react-&-webpack.html) and then adding externals to reference them.  This felt a bit dirty to me.  [Other guides](https://www.smashingmagazine.com/2020/05/typescript-modern-react-projects-webpack-babel/) added extra features or fancier loaders such as [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader).  Not that there is anything wrong with that but I was looking for the most vanilla and basic experience here.

There are also obviously a lot of fantastic automated project generators out there that work as well, such as [nextjs](https://nextjs.org/), or [create react app](https://create-react-app.dev/) but I rolling up my sleeves and doing everything myself here.

Initially I followed [followed this guide for getting setup with just react and webpack](https://www.valentinog.com/blog/babel/), which gives a bare minimum for getting up and going with just those two.

Next came steps for adding in Typescript.  HtmlWebPackPlugin defaults to using 'index.js' as the entry point.  This won't work for Typescript.

Let's experiment with changing that.  Changing the name of index.js to index.jsx broke it, so I had to add the following to the webpack.config.js:

    entry: './src/index.jsx',
    

This tested the ground for using a index.tsx file instead.  

Next came actually adding Typescript.  First running this to install modules:

    npm install --save-dev typescript ts-loader

Then changing swapping out the jsx loader for the [ts-loader](https://github.com/TypeStrong/ts-loader):

    {
        test: /\.ts(x?)$/,
        exclude: /node_modules/,
        use: [
            {
                loader: "ts-loader"
            }
        ]
    }

Also rename all *.jsx/*.js files to *.tsx.

Next you'll get `Can't resolve './App' in '/Users/alex/projects/hello-world/webpack-demo/src'` Add 'resolve' section to webpack.config.js to fix that.

    resolve: {
        extensions: [".ts", ".tsx"]
    },

After this, you'll get error 'The 'files' list in config file 'tsconfig.json' is empty.'  Adding a tsconfig.json fixed that:

    {
        "compilerOptions": {
          "jsx": "react",
          "module": "commonjs",
          "noImplicitAny": true,
          "outDir": "./dist/",
          "preserveConstEnums": true,
          "removeComments": true,
          "sourceMap": true,
          "target": "es5"
        }
    }

But as it turns out, we'll need to resolve 'js' and 'jsx' files as well.  Otherwise you'll get an error similar to `Error: Can't resolve 'object-assign' in '/Users/alex/projects/hello-world/webpack-demo/node_modules/react-dom/cjs'`.  Let's add that now:

    resolve: {
        extensions: [".ts", ".tsx", ".js", ".jsx"]
    },

And guess what this gives us?  Another error.  This time `Could not find a declaration file for module 'react'. '/Users/alex/projects/hello-world/webpack-demo/node_modules/react/index.js' implicitly has an 'any' type.`  This is good, things are getting close to working.  Typescript is now just being nit-picky.  Some people will tell you to disable that error by setting `noImplicitAny` to `false`.  Let's not do that.  Let's take the hard path.   Let's install those type definitions:

    npm install --save-dev @types/react @types/react-dom
    
After this,  `Module '"/Users/alex/projects/hello-world/webpack-demo/node_modules/@types/react/index"' can only be default-imported using the 'esModuleInterop' flag`.  We're going to set that flag to true for now (in tsconfig.json).  Here is a [post with more detail](https://stackoverflow.com/questions/56238356/understanding-esmoduleinterop-in-tsconfig-file).

And there we have it!  We now have a working build setup with all the aforementioned libraries.  Link to the final version [here](https://github.com/ajkovar/webpack-ts-react).