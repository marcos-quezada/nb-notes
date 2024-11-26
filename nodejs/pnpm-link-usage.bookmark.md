# Link and Test Local Packages with pnpm link (serko.dev)

<https://serko.dev/post/pnpm-link-usage>

## Description

When you're developing a JavaScript library, it often needs to be tested in a real project. pnpm link is an extremely useful tool that allows you to add your locally developed package to other projects, facilitating easy testing and debugging. This article will detail how to use pnpm link, including its two main linking modes, and address some potential issues with pnpm link.

## Tags

#nodejs #pnpm

## Content

# Link and Test Local Packages with pnpm link {#link-and-test-local-packages-with-pnpm-link .reader-title}

------------------------------------------------------------------------

When you\'re developing a JavaScript library, it often needs to be tested in a real project. [pnpm link](https://pnpm.io/cli/link){rel="nofollow"} is an extremely useful tool that allows you to add your locally developed package to other projects, facilitating easy testing and debugging. This article will detail how to use pnpm link, including its two main linking modes, and address some potential issues with pnpm link.

## [Global Link](#global-link)

Global Link lets you \"publish\" a library to your local global environment, making it easy to add it to any other local project.

#### [Step 1: \"Publish\" the library in the Global Environment](#step-1-publish-the-library-in-the-global-environment)

#### [Step 2: Add the Global Link Package to a Project](#step-2-add-the-global-link-package-to-a-project)

`my-lib` here is the name of the package, not the path.

### [Unlink Globally](#unlink-globally)

To unlink **all projects** from `my-lib`, execute `pnpm remove --global my-lib` from any location in the system.

### [Unlink from an Individual Project](#unlink-from-an-individual-project)

If `my-app`\'s `package.json` already includes `my-lib` as a dependency, use the following command to unlink:

If `my-lib` is not included in `package.json`, you need to manually remove the related files from `node_modules`.

### [Issues with Global Binary](#issues-with-global-binary)

It\'s worth noting that there are currently issues with the Global binary feature of `pnpm link`. According to the official [documentation](https://pnpm.io/cli/link#add-a-binary-globally){rel="nofollow"}, `pnpm link -g` should support packages with a `bin` section, allowing their binaries to be executed anywhere in the system. However, since May 2022, users have [reported](https://github.com/pnpm/pnpm/issues/4761){rel="nofollow"} that this feature does not work properly in `pnpm 7`, and the problem persists in the latest version `8.12.1`.

Therefore, if your package includes a `bin` section, you might not be able to use the `link -g` command to run its binary directly across the system. The current workaround is to add such packages to a specific project and execute the binary within that project.

## [Directory Link](#directory-link)

Directory Link allows you to directly link a local package to another project, instead of through the global environment.

#### [Method 1: Link the library in the target project](#method-1-link-the-library-in-the-target-project)

#### [Method 2: Link from the library to the target project](#method-2-link-from-the-library-to-the-target-project)

#### [Cancel Directory Link](#cancel-directory-link)

Whether using Method 1 or 2 for Directory link, unlinking is done in the target project:

## [Summary](#summary)

`pnpm link` is similar to `npm link` and `yarn link`, offering a convenient way to link packages in the local environment. While the official pnpm documentation provides basic usage instructions, it may not thoroughly explain practical application, detail handling, and specific use cases. This article aims to fill these gaps, offering a more comprehensive guide on using `link` for developers using `pnpm`.

Currently, certain features of `pnpm link`, like the Global binary, have some issues. These problems can inconvenience developers relying on specific functionalities. As pnpm continues to update and improve, we look forward to these issues being resolved in future versions.

As pnpm evolves, this article will be timely updated to provide the latest information, hoping to assist pnpm developers in achieving a more efficient development workflow.
