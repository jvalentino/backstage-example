# [Backstage](https://backstage.io)



# Developer Setup



## (1) Configure the Auth provider 

Either by creating your own Github OAuth app of asking me for the credentials to mind:

Register a new auth app

![](https://backstage.io/assets/images/gh-oauth-6ba1157307d9e1a95301a49e9ee1b05b.png)

Since you don't want to store these secrets in source control, set them in your profile, for example ~/.zshrc file like this

```bash
BACKSTAGE_EXAMPLE_GIT_CLIENT_ID=SOME ID
BACKSTAGE_EXAMPLE_GIT_SECRET=SOME SECRET
```

When you run the app, you will see this:

![sign-in](./wiki/sign-in.png)

![authorize-app](./wiki/authorize-app.png)

## (2) Make sure the tests work

I generally leave the test watcher on using `yarn test`, noting that getting my current version of VS Code + Jest to successfully work was a pain that I gave up on. As such I added settings to have them not run on save, which you should comment out if you got this to work:

.vscode/settings.json

```json
{
    "jest.runMode": "on-demand"
}
```



To start the app, run:

```sh
yarn install
yarn dev
```



# How this project was created

You need to have a specific version of node installed for the setup to work so be prepared to 

## (1) Install NVM

```shell
brew install nvm
```

## (2) Install the appropriate version of Node you need

```bash
nvm install 20
```

## (3) So that you can create the new template application:

```bash
npx @backstage/create-app@latest
```

...where you will be prompted to give it an application name that will result in that directory being created.

## (4) Database Setup (Local)

 However, backstage requires a database for data storage, so from a local perspective a Postgres database container was created using docker compose:

docker-compose.yaml

```yaml
version: "3.8"
services:
  db:
    container_name: backstage-db
    image: postgres:14.1-alpine
    restart: always
    environment:
      - POSTGRES_USER=backstage
      - POSTGRES_PASSWORD=backstage
      - POSTGRES_DB=backstage
    ports:
      - "5432:5432"
    volumes:
      - ./docker-data/postgres:/var/lib/postgresql/data
```

It is further set to store the data in a directory (that is git ignored) in the project so you don't lose data locally.

This is how you run the local db

```bash
docker-compose up -d
```

You then have to add the Postgres plugin through yarn:

Reference: https://backstage.io/docs/getting-started/config/database

```bash
yarn --cwd packages/backend add pg
```

The result though is that you now have to modify app-config.yaml with the database connection information:

```yaml
backend:
  database:
    client: pg
    connection:
      host: localhost
      port: 5432
      user: backstage
      password: backstage
```

You then validate that this works using `yarn dev`, and then validating using a Postgres client that backstage schemas were created:

![validate-schema](./wiki/validate-schema.png)



## (5) Authentication (Github as an Example)

Reference: https://backstage.io/docs/getting-started/config/authentication

Register a new auth app

![](https://backstage.io/assets/images/gh-oauth-6ba1157307d9e1a95301a49e9ee1b05b.png)

Since you don't want to store these secrets in source control, set them in your profile, for example ~/.zshrc file like this

```bash
BACKSTAGE_EXAMPLE_GIT_CLIENT_ID=SOME ID
BACKSTAGE_EXAMPLE_GIT_SECRET=SOME SECRET
```

Update app-config.yaml with the ID and secret using their env vars:

```yaml
auth:
  # see https://backstage.io/docs/auth/ to learn about auth providers
  environment: development
  providers:
    github:
      development:
        clientId: ${BACKSTAGE_EXAMPLE_GIT_CLIENT_ID}
        clientSecret: ${BACKSTAGE_EXAMPLE_GIT_SECRET}
```

Open `packages/app/src/App.tsx` and below the last `import` line, add:

packages/app/src/App.tsx

```typescript
import { githubAuthApiRef } from '@backstage/core-plugin-api';
import { SignInPage } from '@backstage/core-components';
```

Search for `const app = createApp({` in this file, and below `apis,` add:

packages/app/src/App.tsx

```tsx
components: {
  SignInPage: props => (
    <SignInPage
      {...props}
      auto
      provider={{
        id: 'github-auth-provider',
        title: 'GitHub',
        message: 'Sign in using GitHub',
        apiRef: githubAuthApiRef,
      }}
    />
  ),
},
```



Run the app:

```
yarn dev
```

You will now see this:

![sign-in](./wiki/sign-in.png)

![authorize-app](./wiki/authorize-app.png)

