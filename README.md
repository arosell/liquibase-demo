# liquibase-demo

This is a simple Liquibase demo which deploys a test infrastructure with Docker and deploys some test changesets.

## Run

### Build & Run

Create a demo database using Docker Compose:

```sh
docker-compose up -d
```

### Liquibase Demo

After starting up the test database with `docker-compose up -d`, the database is initialized
with [database/initialize.sql]. That script creates table *tab1* with two columns. The database
state is equal to [liquibase/db.changelog-1.0.xml], but the Liquibase changelog table does not
exist yet.

We can run the `liquibase update` command to create the changelog table and deploy changesets
[liquibase/db.changelog-1.1.sql] and [liquibase/db.changelog-1.2.sql] in one step:

```sh
docker run --rm -v $(pwd)/liquibase:/liquibase/changelog --network=liquibase-demo --workdir="/liquibase/changelog" \
  liquibase/liquibase:4.4 --defaultsFile=/liquibase/changelog/liquibase.properties \
  update
```

To avoid errors when running [liquibase/db.changelog-1.0.xml] while table *tab1* does already
exist, the following precondition is added to the Liquibase changeset:

```xml
<preConditions onFail="MARK_RAN">
    <not>
        <tableExists tableName="tab1"/>
    </not>
</preConditions>
```

Alternatively, we could add a *context* to the legacy database changesets and run the Liquibase update command with `--context=non-legacy`.

### Deconstruct Demo Infrastructure

```sh
docker-compose down
```

## Further Topics

When using Liquibase on an existing database the initial Liquibase Changelog File can be
generated using the following command:

```sh
docker run --rm -v $(pwd)/liquibase:/liquibase/changelog --network=liquibase-demo \
  liquibase/liquibase:4.4 --defaultsFile=/liquibase/changelog/liquibase.properties \
  generate-changelog
```

Alternatively, the Liquibase command can be run on an interactive Shell within the Docker Container:

```sh
docker run -it --rm -v $(pwd)/liquibase:/liquibase/changelog --network=liquibase-demo liquibase/liquibase:4.4 sh
cd /liquibase/changelog
liquibase --defaultsFile=/liquibase/changelog/liquibase.properties generate-changelog
```

As soon as the Liquibase changelog file exists, next step is to create and populate the Liquibase changelog table:

```sh
docker run --rm -v $(pwd)/liquibase:/liquibase/changelog --network=liquibase-demo --workdir="/liquibase/changelog" \
  liquibase/liquibase:4.4 --defaultsFile=/liquibase/changelog/liquibase.properties \
  changelog-sync
```

## References

- [Liquibase - Docker Container](https://hub.docker.com/r/liquibase/liquibase)
- [Liquibase Reference - Config Properties](https://docs.liquibase.com/workflows/liquibase-community/creating-config-properties.html)
- [Liquibase - Best Practices](https://www.liquibase.org/get-started/best-practices)
- [Liquibase - Advantages of XML changelogs over SQL changelogs](https://www.liquibase.org/blog/using-xml-changelogs-liquibase)
- [Liquibase - How to set up Liquibase with an Existing Project and Multiple Environments](https://docs.liquibase.com/workflows/liquibase-community/existing-project.html)