# Liquibase-Maven Docker image

This Docker image contains Liquibase and Maven.
Forked from [https://github.com/liquibase/docker](official Liquibase Docker image).

## Run Liquibase

Define the Maven goal (e.g. liquibase:update) at container start.

Example:

```sh
docker run --rm -v $(pwd)/liquibase:/liquibase/changelog --network=liquibase-demo --workdir="/liquibase/changelog" liquibase_maven liquibase:update
```
