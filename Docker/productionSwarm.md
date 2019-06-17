# Docker Swarm in Produciton

## Secrets

Working with Docker secrets to turn my `.env` file variables into **swarm secrets**. 

1. The Dockerfile for the image needs to contain an `ENTRYPOINT` script which will take the environment variable string, search for any variable set to that name with `_FILE` suffixed, and will set the contents of the file to the apporpriate environment variable.

```bash
# usage: file_env VAR [DEFAULT]
#    ie: file_env 'XYZ_DB_PASSWORD' 'example'
# (will allow for "$XYZ_DB_PASSWORD_FILE" to fill in the value of
#  "$XYZ_DB_PASSWORD" from a file, especially for Docker's secrets feature)
file_env() {
	local var="$1"
	local fileVar="${var}_FILE"
	local def="${2:-}"
	if [ "${!var:-}" ] && [ "${!fileVar:-}" ]; then
		echo >&2 "error: both $var and $fileVar are set (but are exclusive)"
		exit 1
	fi
	local val="$def"
	if [ "${!var:-}" ]; then
		val="${!var}"
	elif [ "${!fileVar:-}" ]; then
		val="$(< "${!fileVar}")"
	fi
	export "$var"="$val"
	unset "$fileVar"
}
```

The `ENTRYPOINT` in the dockerfile need to call the above function for each environment variable I wish to pass in with secrets.

```bash
file_env 'MONGODB_USER'
file_env 'MONGODB_PASS'
```

2. The compose or stack.yml file and how to use the secrets:
```yml
version: 3.7
services:
  app:
  # assign the secrets to be consumed by this service
    secrets:
      - mongodb_user
      - mongodb_pass
    # Use of the secrets
    environment:
      MONGODB_USER: '/run/secrets/mongodb_user'
      MONGODB_PASS: '/run/secrets/mongodb_pass'
# Declare the secrets in the stack
secrets:
  mongodb_user:
    external: true
  mongodb_pass:
    external: true
```

3. Create the secrets in the `swarm`. To initialize docker swarm if not already done so run: `docker swarm init`. It may prompt for the ip address to use with `--advertise-addr`
```bash
docker swarm init --advertise-addr <ip-address>
```

The secrets can be created with either a file or from STDOUT.
```bash
docker secret create <name> <file>
//OR
echo "some-secret" | docker secret create <name> -
```

Both the above steps have an issue with creating the secrets without leaving behind a record of their values; either as a text file or in the bash history log.

I can get around the above issue by setting the DOCKER_HOST env in the shell session to that the docker commands are run against the remote docker engine. 
Run:
```bash
export DOCKER_HOST=ssh://<username>@<host>
```
