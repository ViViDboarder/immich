---
sidebar_position: 3
---


# Recommended installation

This is the recommended installation method for production use.

### Step 1 - Download the required files

From a directory of your choice (e.g. `./immich-app`) run the following commands

```bash title="Get docker-compose.yml file"
wget https://raw.githubusercontent.com/immich-app/immich/main/docker/docker-compose.yml
```

```bash title="Get .env file"
wget -O .env https://raw.githubusercontent.com/immich-app/immich/main/docker/.env.example
```

### Step 2 - Populate the .env file with custom values

<details>

<summary>Example <code>.env</code> content</summary>

```bash
###################################################################################
# Database
###################################################################################

DB_HOSTNAME=immich_postgres
DB_USERNAME=postgres
DB_PASSWORD=postgres
DB_DATABASE_NAME=immich

# Optional Database settings:
# DB_PORT=5432

###################################################################################
# Redis
###################################################################################

REDIS_HOSTNAME=immich_redis

# Optional Redis settings:
# REDIS_PORT=6379
# REDIS_DBINDEX=0
# REDIS_PASSWORD=
# REDIS_SOCKET=

###################################################################################
# Upload File Config
###################################################################################

UPLOAD_LOCATION=absolute_location_on_your_machine_where_you_want_to_store_the_backup

###################################################################################
# Log message level - [simple|verbose]
###################################################################################

LOG_LEVEL=simple

###################################################################################
# JWT SECRET
###################################################################################

# This JWT_SECRET is used to sign the authentication keys for user login
# You should set it to a long randomly generated value
# You can use this command to generate one: openssl rand -base64 128
JWT_SECRET=kWPdavjCECB0yoXgUHA/vpwpIKdCi/4ODVLIOe9WIi6AQlFfjWEuIVhWT3DtJE+T
CTckJnpwGgSK5AoqD+A8DZKsHCRdfVnlQIVqqmyR8isZTcxL5DWYQUSDRzyOO5OA
ZRUTE63FxiYhrRoe/y1yr5mV1osGy6mm6NZW8T2Tjwc=

###################################################################################
# Reverse Geocoding
####################################################################################

# DISABLE_REVERSE_GEOCODING=false

# Reverse geocoding is done locally which has a small impact on memory usage
# This memory usage can be altered by changing the REVERSE_GEOCODING_PRECISION variable
# This ranges from 0-3 with 3 being the most precise
# 3 - Cities > 500 population: ~200MB RAM
# 2 - Cities > 1000 population: ~150MB RAM
# 1 - Cities > 5000 population: ~80MB RAM
# 0 - Cities > 15000 population: ~40MB RAM

# REVERSE_GEOCODING_PRECISION=3

####################################################################################
# WEB - Optional
####################################################################################

# Custom message on the login page, should be written in HTML form.
# For example PUBLIC_LOGIN_PAGE_MESSAGE="This is a demo instance of Immich.<br><br>Email: <i>demo@demo.de</i><br>Password: <i>demo</i>"

PUBLIC_LOGIN_PAGE_MESSAGE="My Family Photos and Videos Backup Server"
```

</details>

* Populate custom database information if necessary.
* Populate `UPLOAD_LOCATION` as prefered location for storing backup assets.
* Populate a secret value for `JWT_SECRET`, you can use the command below to generate a secured key

```bash title="Command to generate secure JWT_SECRET key"
openssl rand -base64 128
```

### Step 3 - Start the containers

```bash title="Start the containers using docker compose command"
docker-compose up -d # or `docker compose up -d` based on your docker-compose version
```

:::tip
For more information on how to use the application, please refer to the [Post Installation](/docs/usage/post-installation) guide.
:::