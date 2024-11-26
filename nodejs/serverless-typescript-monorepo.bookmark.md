# Serverless TypeScript monorepo (insify.medium.com)

<https://insify.medium.com/serverless-with-typescript-in-a-monorepo-4-ways-to-make-it-work-4ea0b8f43608>

## Description

Several months ago our team has embarked on a new project. An exciting opportunity to choose a modern technology stack that suits you! So we did. The detail is that you need to make this new stack…

## Tags

#nodejs #ts #serverless

## Content

# Serverless with Typescript in a monorepo? 4 ways to make it work. {#serverless-with-typescript-in-a-monorepo-4-ways-to-make-it-work. .reader-title}

Insify

------------------------------------------------------------------------

[](https://insify.medium.com/?source=post_page---byline--4ea0b8f43608--------------------------------){rel="noopener follow"}

![Insify](https://miro.medium.com/v2/resize:fill:88:88/1*myppKIP6NrdFAzyXW1NpVQ.png){testid="authorPhoto" loading="lazy" height="44" width="44"}

## New journey {#486d}

Several months ago our team has embarked on a new project. An exciting opportunity to choose a modern technology stack that suits you! So we did. The detail is that you need to make this new stack working smoothly, and your choices don't backfire on you. Sometimes it is not trivial.

For example, you decided to manage all your projects in a single repository. Which also means managing their dependencies. At the same time, you opted for Serverless architecture. Your Lambdas need to be packaged with just enough libraries: they can run but remain slim. You'd think all should work out of the box. Not necessarily.

With a bit of experimentation, you can get it right. We'd like to save you some time, as we have found several alternative solutions to the Lambda packaging issue.

## Stack {#f2ed}

We opted for technologies that allow you to develop and scale your system quickly.

-   [AWS as a Cloud Provider]{#1ca2}
-   [[Serverless](https://www.serverless.com/){rel="noopener ugc nofollow"} framework to manage the backend services and infrastructure]{#0ae8}
-   [NodeJS on the backend]{#1f08}
-   [React on the frontend (no mobile yet)]{#0ec9}
-   [Typescript as a primary language everywhere]{#0c35}
-   [Yarn as a package manager]{#351f}
-   [[Monorepo](https://en.wikipedia.org/wiki/Monorepo){rel="noopener ugc nofollow"} as a way to manage the codebase.]{#fe01}

It appeared that a combination of these technologies requires some extra care.

## Show me code {#7c27}

A real code example makes it easier to picture the point. [Here is a simple app](https://github.com/insify/serverless-typescript-monorepo-example/tree/master){rel="noopener ugc nofollow"} to create polls, vote, and see the results (it's not production-grade).

The app consists of 4 parts:

-   [Infrastructure --- a DynamoDB table that stores polls]{#800e}
-   [Service --- business logic and API implemented as AWS Lambda and API Gateway]{#7f0c}
-   [App --- a react application]{#bac2}
-   [Model --- shared code between service and app (In the real world you rather go with OpenAPI/JSON schema)]{#6b31}

Around 100 lines of YAML, and Serverless can create the infrastructure and deploy the backend. Marvelous!

And... it doesn't work.

You realize why by exploring a packaged Lambda function: it misses all the runtime dependencies. The service depends on the application's model and the `shortid` library:

    package.json # All code examples assume poll/service directory"dependencies": {
      "@poll/model": "1.0.0",
      "shortid": "^2.2.15"
    }

A deployed Lambda does not have any dependency:

<figure>

</figure>

<figure>

</figure>

Let's find out why!

## Looking for a missing piece {#7754}

A standard practice in a monorepo is to use [Yarn workspaces](https://classic.yarnpkg.com/en/docs/workspaces/){rel="noopener ugc nofollow"}. In this case, the dependencies get hoisted, so there is only one big fat `node_modules` folder, at the project root. In our case, the `service` workspace has an empty `node_modules`. Maybe it confuses Serverless?

## Attempt 1: Disable hoisting {#7ba7}

Yarn Workspaces offer the [nohoist](https://classic.yarnpkg.com/blog/2018/02/15/nohoist/){rel="noopener ugc nofollow"} option: you can specify the dependencies you want to reside in a local `node_modules` . It should help:

    package.json"private": true, 
    +  "workspaces": { 
    +    "nohoist": [ 
    +      "@poll/model",
    +      "shortid"
    +    ] 
    +  },

Good news: now the Lambda package includes the `shortid` library. Bad news: it still misses the model. It's weird because the model is present in the `node_modules` directory of the service:

    ls node_modules/ 
    @poll  shortidll node_modules/@poll 
    model -> ../../../model

Aha, it's a symlink! And it doesn't get packaged.

One more issue, by the way, is that `shortid` depends on another library `nanoid` that is also missing.

Clearly, disabling hoisting does not work by itself.

## We got it to work, but... {#cb5f}

Disabling hoisting completely

    package.json
    "private": true, 
    +  "workspaces": { 
    +    "nohoist": [ 
    +      "**"
    +    ] 
    +  },

and employing [Yalc](https://www.npmjs.com/package/yalc){rel="noopener ugc nofollow"} to copy dependencies into `node_modules` instead of using symlinks does the trick. You can see the result [in this branch](https://github.com/insify/serverless-typescript-monorepo-example/tree/experiment1/disable-hoisting){rel="noopener ugc nofollow"}.

This is quite suboptimal. Having multiple workspaces with a big fat `node_modules` makes your builds slower. And you need to build tooling that copies around shared modules every time they get changed (later we found the way to avoid copying and make symlinks work--- see Attempt 2). Is there a more nifty solution?

Fortunately, Serverless has plenty of useful plugins. Let's have a look.

## Attempt 2: serverless-plugin-monorepo {#cba0}

It's easy to google this plugin, and it looks [promising](https://www.npmjs.com/package/serverless-plugin-monorepo){rel="noopener ugc nofollow"}:

> *A Serverless plugin design to make it possible to use Serverless in a Javascript mono repo with hoisted dependencies, e.g. when using Yarn Workspaces.*
>
> *This plugin alleviates the need to use* [*nohoist*](https://yarnpkg.com/blog/2018/02/15/nohoist/){rel="noopener ugc nofollow"} *functionality by creating symlinks to all declared dependencies.*

The plugin has created symlinks to all the necessary libraries, but these libraries didn't make it to Lambda's bundle. Same problem as in the previous case.

After some digging, it appears that the culprit is `serverless-plugin-typescript` that transpiles Typescript to Javascript before packaging Lambdas. When the plugin is active, Serverless ignores the symlinked dependencies. The workaround would be to not use the plugin and [call](https://github.com/insify/serverless-typescript-monorepo-example/tree/experiment2/sls-monorepo-plugin){rel="noopener ugc nofollow"} [`tsc`](https://github.com/insify/serverless-typescript-monorepo-example/tree/experiment2/sls-monorepo-plugin){rel="noopener ugc nofollow"}[manually](https://github.com/insify/serverless-typescript-monorepo-example/tree/experiment2/sls-monorepo-plugin){rel="noopener ugc nofollow"}. In a nutshell:

    serverless.yml
    plugins:
    -  - serverless-plugin-typescript 
    +  - serverless-plugin-monorepo# Exclude Typescript source and other dev-related files 
    +package: 
    +  exclude: 
    +    - src/*.ts 
    +    - node_modules/.bin/** 
    +    - '*.json'package.json:
    +  "scripts": { 
    +    "package": "tsc && serverless package", 
    +    "deploy": "tsc && serverless deploy", 
    +    "deploy:fn": "tsc && serverless deploy function"
    +  }tsconfig.json:
    +  "compile": "tsc",

Then:

    cd poll/service
    sls package
    ls -lh .serverless40K poll.zip

Looks OK, still can be better. Please also note that the package is the same for all functions. It means that for each Lambda function it will deploy the other Lambdas and their dependencies as well. In this case, every Lambda would weight only 40KB. In real life it can grow much bigger (just add `axios` as a dependency and see happens). It would be nice to find a way to package only code that a function needs.

Let's keep digging!

## Attempt 3: serverless-webpack-plugin {#4860}

Another popular way to package Lambdas is by using the [serverless-webpack plugin](https://www.npmjs.com/package/serverless-webpack){rel="noopener ugc nofollow"}. It transpiles Typescript with Babel, no need in `serverless-plugin-typescript`.

Additionally, the plugin can package each function only with the dependencies it uses, resolving the size issue that we recently mentioned.

Let's add a couple of lines to serverless.yml:

    serverless.yml
    + package:
    +   individually: true

And build it:

    sls packagels -lh .serverless/poll.zip1.2M create.zip
    1.2M getPoll.zip
    1.2M getSummary.zip
    1,2K vote.zip

Oops, 1.2 megabytes per function doesn't look good. Apparently, `aws-sdk` creeps in, even if you explicitly exclude it from the bundle. Fortunately, there is a [workaround](https://github.com/serverless-heaven/serverless-webpack/issues/306#issuecomment-420426295){rel="noopener ugc nofollow"}.

A bit of sorcery in `webpack.config.js` (the full version is [here](https://github.com/insify/serverless-typescript-monorepo-example/tree/experiment3/webpack-plugin){rel="noopener ugc nofollow"}), and the result is quite good:

    sls package
    ls -lh .serverless/13K create.zip
    5,8K getPoll.zip
    5,8K getSummary.zip
    5,8K vote.zip

This is essentially what we want! And it can be even better.

## Attempt 4: serverless-plugin-optimize {#50a9}

Here comes[`serverless-plugin-optimize`](https://www.npmjs.com/package/serverless-plugin-optimize){rel="noopener ugc nofollow"}

> *Bundle with Browserify, transpile and minify with Babel automatically to your NodeJS runtime compatible JavaScript.*

It doesn't require any additional configuration and works seamlessly with `serverless-plugin-typescript` :

    serverless.json
    plugins: 
       - serverless-plugin-typescript 
    +  - serverless-plugin-optimize   +package: 
    +  individually: true # should optimize each function

and the result:

    sls package 
    cd poll/service
    sls package
    ls -lh .serverless6,8K create.zip
    3,0K getPoll.zip
    3,0K getSummary.zip
    3,0K vote.zip

The total size of the functions is only 16K. You can see the effort [in this pull request](https://github.com/insify/serverless-typescript-monorepo-example/pull/1){rel="noopener ugc nofollow"}**.** Only 4 lines of code resolve the packaging issue and ensure our Lambdas are tiny. Neat!

## Conclusion {#3d6d}

When you meet a technical issue, Google is your big friend. Sometimes though you find an approach works for others, and not for you because of a little detail that you need to discover. You can also find a solution quickly, but is it the best one?

In this post we explored several ways of packaging Lambda functions when you deal with Serverless, NodeJS and Typescript in a monorepo. 3 out of 4 required a workaround to start working! Clearly, some are more efficient than others.

Here they are, ranked by our preference:

1.  [`serverless-plugin-optimize` [nails it](https://github.com/insify/serverless-typescript-monorepo-example/tree/experiment4/serverless-optimize-plugin){rel="noopener ugc nofollow"}! 👍]{#2184}
2.  [`serverless-plugin-webpack `a strong contender. [Requires more configuration](https://github.com/insify/serverless-typescript-monorepo-example/tree/experiment3/webpack-plugin){rel="noopener ugc nofollow"}, and the package sizes are a bit bigger. 👍]{#aabc}
3.  [`serverless-plugin-monorepo` [needs a workaround](https://github.com/insify/serverless-typescript-monorepo-example/tree/experiment2/sls-monorepo-plugin){rel="noopener ugc nofollow"} to transpile Typescript, and the package size is not optimal. 😐]{#d07c}
4.  [Disable hoisting and use `yalc`. [Headache with dependencies](https://github.com/insify/serverless-typescript-monorepo-example/tree/experiment1/disable-hoisting){rel="noopener ugc nofollow"}, package size is not optimal. 👎]{#c9a5}

If you have another solution that works for you, please let us know!
