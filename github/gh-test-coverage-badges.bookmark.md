# Adding test coverage badge on Github without third party services (hackernoon.com)

<https://hackernoon.com/adding-test-coverage-badge-on-github-without-using-third-party-services>

## Description

It’s easy to add test coverage on GitLab using the built-in feature.

## Tags

#gh #test

## Content

# Adding Test Coverage Badge on GitHub Without Using Third-party Services {#adding-test-coverage-badge-on-github-without-using-third-party-services .reader-title}

Anton Yakutovich

------------------------------------------------------------------------

It's easy to [add test coverage on GitLab](https://docs.gitlab.com/ee/ci/yaml/index.html?ref=hackernoon.com#coverage){rel="noopener noreferrer ugc"} using the built-in feature. One line in `.gitlab-ci.yml` to rule them all:

    test:
      coverage: /\d+.\d+ \% covered/

On the opposite side, GitHub doesn't provide an option to add the test coverage badge. There are many third-party services for this purpose: [codeclimate](https://codeclimate.com/?ref=hackernoon.com){rel="noopener noreferrer ugc"}, [codecov](https://about.codecov.io/?ref=hackernoon.com){rel="noopener noreferrer ugc"}, [codacy](https://www.codacy.com/?ref=hackernoon.com){rel="noopener noreferrer ugc"}, [coveralls](https://coveralls.io/?ref=hackernoon.com){rel="noopener noreferrer ugc"}. It's not a problem for an open-source project to use these services, cause they mostly provide a free tier for the open-source.

It hurts if you want to add the coverage badge to a private repository. All 3rd party options are paid. Samurai has two choices:

1.  Use a service to draw a badge based on provided JSON ([shields.io](http://shields.io/?ref=hackernoon.com){rel="noopener noreferrer ugc"});
2.  Generate a badge during CI/CD run and store it in the repository.

Both options work. In the first case, you\'ll have to use an intermediate GitHub Gist, where you\'ll send the [fresh coverage as a JSON file](https://dev.to/thejaredwilcurt/coverage-badge-with-github-actions-finally-59fa?ref=hackernoon.com){rel="noopener noreferrer ugc"}. Not an elegant approach.

You must commit the SVG image to the repository in the second case. I\'ve often seen recipes on the Internet where the badge was committed directly to the main branch. I think this is a bad practice: you\'ll clog up the history with garbage commits, which are useless by nature.

There is a better choice. GitHub provides Pages service, where you can publish a coverage report for example. Technically, you can publish any static set of HTML files placed in a special `gh-pages` branch. Looks like an ideal place for storing badges because it will not clog up the history of the main branch.

Take a look at the snippet with URLs below. If you are using a private repository, you will probably want to make the published report on GitHub Pages closed behind authorization as well. In that case, you won\'t be able to add a badge reference `(1)` inside your README. Instead, provide a link to the RAW file in the repository. GitHub will redirect the request `(2)` to something like `(3)`. The `token` parameter is generated for a closed repository for authorization purposes. The token is temporary --- after several hours the link will become a 🎃. To avoid such behavior you may add a parameter `raw=true` to the file's link:

`https://github.com/user/my-project/raw/gh-pages/badges/jacoco.svg?raw=true`

    https://my-project.pages.github.io/badges/coverage.svg (1) GitHub Pages link
    https://github.com/user/my-project/raw/gh-pages/badges/coverage.svg (2) RAW link
    https://raw.githubusercontent.com/user/my-project/gh-pages/badges/coverage.svg?token=GHSAT0AAAAAABTVGF2OW2264AQOCSDOKKHGYXPQLUA (3) Permalink to the file in the private repo

Let\'s put all the requirements together:

1.  You need to generate a badge using a command-line tool or a GitHub Action;
2.  It's better to place the resulting badge in the `gh-pages` branch;
3.  In order to display the picture in the README even in a closed repository, you must use a direct link to the file with the `raw=true` parameter.

Practice time! I forked **[spring-boot-realworld-example-app](https://github.com/drakulavich/spring-boot-realworld-example-app?ref=hackernoon.com){rel="noopener noreferrer ugc"}** repository. This is one of the implementations of the Conduit App (Medium.com clone).

1.  We'll use [jacoco-badge-generator](https://github.com/cicirello/jacoco-badge-generator?ref=hackernoon.com){rel="noopener noreferrer ugc"} Action.

    Add a new step to the GitHub [Workflow config](https://github.com/drakulavich/spring-boot-realworld-example-app/blob/ci/coverage_badge/.github/workflows/gradle.yml?ref=hackernoon.com){rel="noopener noreferrer ugc"}:

            - name: Generate JaCoCo Badge
        #      if: ${{ github.ref == 'refs/heads/master' }}
              uses: cicirello/jacoco-badge-generator@v2
              with:
                jacoco-csv-file: build/reports/jacoco/test/jacocoTestReport.csv
                badges-directory: build/reports/jacoco/test/html/badges

We are specifying paths to JaCoCo CSV report and the resulting badge\'s directory. Uncomment the second line if you'd like to execute the step only in the main branch.

2.  The next step is to publish the report along with badges. I choose [github-pages-deploy-action](https://github.com/JamesIves/github-pages-deploy-action?ref=hackernoon.com){rel="noopener noreferrer ugc"}.

<!-- -->

        - name: Publish coverage report to GitHub Pages
    #      if: ${{ github.ref == 'refs/heads/master' }}
          uses: JamesIves/github-pages-deploy-action@v4
          with:
            folder: build/reports/jacoco/test/html

We need to set the path to the HTML report. Just a reminder, the generated badge will also be inside this directory. After the commit you'll need to open the repository **Settings → Pages** and select the branch you want to publish the site from:

![Configure GitHub Pages](https://hackernoon.imgix.net/images/2hVuiN1gfbdO9OXUxjCttPNETq73-jb937zu.png?auto=format&fit=max&w=3840){old-src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7" loading="lazy" nimg="intrinsic" decoding="async" srcset="https://hackernoon.imgix.net/images/2hVuiN1gfbdO9OXUxjCttPNETq73-jb937zu.png?auto=format&fit=max&w=1920 1x, https://hackernoon.imgix.net/images/2hVuiN1gfbdO9OXUxjCttPNETq73-jb937zu.png?auto=format&fit=max&w=3840 2x"}

3.  The last piece to add is the link to the badge in README. Don\'t forget about the `raw=true` parameter, so that the picture will be displayed for private repositories as well:

        [![Test Coverage](https://cdn.hackernoon.com/images/2hVuiN1gfbdO9OXUxjCttPNETq73-2022-08-08T22:19:48.257Z-cl6lbgrz6002p0as65v8sfgja)](https://drakulavich.github.io/spring-boot-realworld-example-app/)

Hooray! All steps have been completed, you can [check the result](https://github.com/drakulavich/spring-boot-realworld-example-app/tree/ci/coverage_badge?ref=hackernoon.com#readme){rel="noopener noreferrer ugc"}.

![Test coverage from JaCoCo](https://hackernoon.imgix.net/images/2hVuiN1gfbdO9OXUxjCttPNETq73-g9a37av.png?auto=format&fit=max&w=3840){old-src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7" loading="lazy" nimg="intrinsic" decoding="async" srcset="https://hackernoon.imgix.net/images/2hVuiN1gfbdO9OXUxjCttPNETq73-g9a37av.png?auto=format&fit=max&w=1920 1x, https://hackernoon.imgix.net/images/2hVuiN1gfbdO9OXUxjCttPNETq73-g9a37av.png?auto=format&fit=max&w=3840 2x"}

By clicking on the badge you'll open the coverage report:

![JaCoCo coverage report published on GitHub Pages](https://hackernoon.imgix.net/images/2hVuiN1gfbdO9OXUxjCttPNETq73-49b37x3.png?auto=format&fit=max&w=3840){old-src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7" loading="lazy" nimg="intrinsic" decoding="async" srcset="https://hackernoon.imgix.net/images/2hVuiN1gfbdO9OXUxjCttPNETq73-49b37x3.png?auto=format&fit=max&w=3840 1x"}

Please share in the comments your favorite GitHub Actions to prepare and publish the badges. Thank you for reading!
