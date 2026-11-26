# karaf-maven-plugin 3.0.10-DEPATCH1

Patched build of `org.apache.karaf.tooling:karaf-maven-plugin` 3.0.10 that works with Maven 3.9.

## Why

`features-create-kar` (`CreateKarMojo`) of karaf-maven-plugin 3.0.x writes a
`maven-metadata-local.xml` with `localCopy=true` next to every SNAPSHOT bundle it packs — directly
into the **local Maven repository**, also for bundles that were only downloaded, never installed.
Maven 3.9 reads that file as a local installation, does not find the installed file and fails with
`Could not find artifact …-SNAPSHOT`. Maven 3.8 ignored the file. Karaf 3.x is no longer
maintained upstream, so there is no official fix.

## What changed

Only the code that the bug needs, everything else is upstream `karaf-3.0.10`:

* `tooling/karaf-maven-plugin/src/main/java/org/apache/karaf/tooling/features/CreateKarMojo.java`
  — when the local repository has no `maven-metadata-local.xml` for a SNAPSHOT bundle, the file is
  generated below the build directory (`target/kar-metadata/<group>/<artifact>/<version>/`) instead
  of the local repository. It is packed into the `.kar` under the same path as before, so the `.kar`
  content does not change. An existing file in the local repository is read as before, never written.
* `tooling/karaf-maven-plugin/pom.xml` — version `3.0.10-DEPATCH1`; a `dependencyManagement` block
  keeps the Karaf runtime modules on the upstream release 3.0.10 (the parent manages them with
  `${project.version}`).

Coordinates are unchanged: `org.apache.karaf.tooling:karaf-maven-plugin:3.0.10-DEPATCH1`.
Its parent `org.apache.karaf.tooling:tooling:3.0.10` comes from Maven Central.

## Build

The checkout may be sparse — only `/pom.xml`, `/tooling/pom.xml`, `/tooling/karaf-maven-plugin/`
and `/etc/appended-resources/` are needed. Build with a JDK 8, like upstream 3.0.x:

```shell
mvn -B -DskipTests -Drat.skip=true -f tooling/karaf-maven-plugin/pom.xml clean install
```

## Deploy

The artifact is not published to Maven Central. Deploy it to your own repository manager, with the
credentials of a `<server>` entry of your Maven `settings.xml`:

```shell
mvn -B -DskipTests -Drat.skip=true -f tooling/karaf-maven-plugin/pom.xml clean deploy \
    "-DaltDeploymentRepository=<server-id>::default::<repository-url>"
```

## Use in a project

Set only the plugin version; the Karaf runtime version stays as it is:

```xml
<plugin>
    <groupId>org.apache.karaf.tooling</groupId>
    <artifactId>karaf-maven-plugin</artifactId>
    <version>3.0.10-DEPATCH1</version>
    <extensions>true</extensions>
</plugin>
```

A local repository that was used with an unpatched 3.0.x plugin may still hold
`maven-metadata-local.xml` files with `localCopy=true` in SNAPSHOT folders that were never
installed; delete those SNAPSHOT folders once.
