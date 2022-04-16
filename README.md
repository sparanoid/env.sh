# env.sh

Dead-simple .env file reader and generator

## Features

- Designed to be used for Next.js app inside a Docker container (General React app should also work)
- Read env file and generate `__env.js` file for runtime client use
- Merge current environment variables passing to it (Useful for Docker images)
- No dependencies (More like a lite version of [react-env](https://github.com/andrewmclagan/react-env)). This is important to keep our container image as small as possible.

## Usage

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

Use it inside Dockerfile:

Install `bash` for your image first (Here operator used in this script does not work with `sh` at the moment. This can be changed in the furture):

```dockerfile
RUN apk add --no-cache bash
```

```dockerfile
RUN chmod +x ./env.sh
ENTRYPOINT ["./env.sh"]
```

## License

 Apache-2.0
