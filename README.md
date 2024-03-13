# [Backstage](https://backstage.io)





# Creation

You need to have a specific version of node installed for the setup to work so be prepared to 

(1) Install NVM

```shell
brew install nvm
```

(2) Install the appropriate version of Node you need

```bash
nvm install 20
```



So that you can create the new template application:

```bash
npx @backstage/create-app@latest
```

...where you will be prompted to give it an application name that will result in that directory being created.

# Runtime



To start the app, run:

```sh
yarn install
yarn dev
```
