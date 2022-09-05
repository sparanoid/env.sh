# env.sh

Dead-simple `.env` file reader and generator

## Features

- Designed to be used for Next.js app inside a Docker container (General React app should also work)
- Read env file and generate `__env.js` file for runtime client use
- Merge current environment variables passing to it (Useful for Docker images)
- No dependencies (More like a lite version of [react-env](https://github.com/andrewmclagan/react-env)). But does not require you to build your project inside the container. The benefit of this method can help you avoid multistage Dockerfile. You can build your Next.js project in GitHub Actions. And only `COPY` built files required for production. With this method your image can be much smaller.

## Usage

Simply copy `env.sh` to the root of your project. And follow the steps.

General usage:

```shell
$ ./env.sh
```

Replacing varaible:

```shell
$ NEXT_PUBLIC_API_BASE=xxx ./env.sh
```

Enviroment variable not in whitelist will be discarded:

```shell
$ BAD_ENV=zzz ./env.sh
```

Change script options:

```shell
$ ENVSH_ENV="./.env.staging" ENVSH_OUTPUT="./public/config.js" ./env.sh
```

Use it with your existing project. Modify your scripts in `package.json`:

```json
"scripts": {
  "dev": "bash env.sh next dev",
  "build": "CI=true bash env.sh next build",
}
```

Create a `utils.js` to use it in runtime client:

```js
// Simplified from:
// https://github.com/andrewmclagan/react-env/blob/master/packages/node/src/index.js
export function env(key = '') {
  if (!key.length) {
    throw new Error('No env key provided');
  }

  if (isBrowser() && window.__env) {
    return window.__env[key] === "''" ? '' : window.__env[key];
  }

  return process.env[key] === "''" ? '' : process.env[key];
}
```

In `_document.js`:

```js
<Head>
  <script src="/__env.js" />
</Head>
```

In `MyComponent.js`:

```js
import { env } from 'utils';
const API_BASE = env('CUSTOM_API_BASE');
```

Now you can run `yarn dev` and see `API_BASE` dynamically injected to your app for client-side rendering.

Use it inside `Dockerfile`:

Install `bash` for your image first (Here operator used in this script does not work with `sh` at the moment. This can be improved in the furture):

```dockerfile
RUN apk add --no-cache bash

# ...other build steps

RUN chmod +x ./env.sh
ENTRYPOINT ["./env.sh"]

# ...custom next.js server if required
CMD ["node", "server.js"]
```

## Options

### `ENVSH_ENV`

Specify env file to read.

Default: `./.env`

### `ENVSH_PREFIX`

Only environment variables with this prefix will be matched.

Default: `NEXT_PUBLIC_`

### `ENVSH_PREFIX_STRIP`

If set to `true`, `ENVSH_PREFIX` will be stripped from the variable name:

```js
window.__env = {
API: "https://openbayes.com/",
DEBUG: "true",
}
```

When set to `false`:

```js
window.__env = {
NEXT_PUBLIC_API: "https://openbayes.com/",
NEXT_PUBLIC_DEBUG: "true",
}
```

Default: `true`

### `ENVSH_PREPEND`

String to be prepended to the output config name. If you change this, you should also change the way to access it in `utils.js`.

Default: `window.__env = {`

### `ENVSH_APPEND`

String to be appended to the output config name.

Default: `}`

### `ENVSH_OUTPUT`

The filename of the output config file.

Default: `./public/__env.js`

### `ENVSH_VERBOSE`

Debug level output.

Default: `false`

## Security Alert

This script is used to read environment variables from a file and inject matched variables for client-side use. So It's not safe to prepend your private API keys or tokens with `ENVSH_PREFIX`. If you do that these variables will be available in `ENVSH_OUTPUT` file and exposed to the clients.

## License

Apache-2.0
