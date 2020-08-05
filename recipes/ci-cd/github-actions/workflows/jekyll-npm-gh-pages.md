# Jekyll with NPM packages on GH Pages
> Run a build with NPM packages and Jekyll then publish to GitHub Pages

This flow comes from DevHints [rstacruz/cheatsheet](https://github.com/rstacruz/cheatsheets) repo. 

It is low-level in working with tools - it may not be efficient compared with using other actions and workflows. But, it is a complete solution, so it could be useful to learn from. The file has been split into pieces so I can focus on using any piece.

- Base setup
    ```yaml
      name: Deploy
      on: 
        push:
          branches:
            master
      
      jobs:
          build:
            runs-on: ubuntu-latest

            steps:
                # ...
    ```
- Checkout step.
    ```yaml
          - uses: actions/checkout@v2
            with:
              persist-credentials: false
    ```
- Steps to setup use cache
    ```yaml
          # https://github.com/actions/cache/blob/master/examples.md#node---yarn
          - name: "Cache: Get yarn cache directory path"
            id: yarn-cache-dir-path
            run: echo "::set-output name=dir::$(yarn cache dir)"

          - name: "Cache: Set up yarn cache"
            uses: actions/cache@v2
            id: yarn-cache
            with:
              path: ${{ steps.yarn-cache-dir-path.outputs.dir }}
              key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
              restore-keys: |
                ${{ runner.os }}-yarn-

          # https://github.com/actions/cache/blob/master/examples.md#ruby---bundler
          - name: "Cache: Set up bundler cache"
            uses: actions/cache@v2
            with:
              path: vendor/bundle
              key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
              restore-keys: |
                ${{ runner.os }}-gems-
    ```
- Setup NodeJS (and Yarn) and Ruby.
    ```yaml
          - name: Use Node.js
            uses: actions/setup-node@v1
            with: { node-version: '12.x' }

          - name: Use Ruby
            uses: actions/setup-ruby@v1
            with: { ruby-version: '2.7.1' }
    ```
- Setup node packages and gems and do the yarn build. NPM could be used here too.
    ```yaml
          - name: Setup dependencies
            run: |
              yarn install --frozen-lockfile
              bundle config path vendor/bundle
              bundle install --jobs 4 --retry 3

          - run: yarn build
    ```
- GitHub pages deploy to main site and to a mirror. This will run on the `jekyll build` command and use the NPM build output from above.
    ```yaml
          - name: "Deploy to gh-pages 🚀"
            uses: JamesIves/github-pages-deploy-action@releases/v3
            with:
              GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
              BRANCH: gh-pages
              FOLDER: _site

          - name: "Deploy to mirror 🚀"
            uses: JamesIves/github-pages-deploy-action@releases/v3
            with:
              ACCESS_TOKEN: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
              REPOSITORY_NAME: rstacruz/devhints-mirror
              BRANCH: gh-pages
              FOLDER: _site
    ```