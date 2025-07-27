# ☕ Maven Tiles - Spotless Tile Example

This project demonstrates how to modularize Maven plugin configurations using [`tiles-maven-plugin`](https://github.com/repaint-io/maven-tiles), specifically by creating a reusable tile for [Spotless](https://github.com/diffplug/spotless).

## 📌 What Is Maven Tile?

**Maven Tiles** is a plugin that lets you extract `build.plugins` configuration into reusable, shareable files (`tile.xml`) called _Tiles_. Other Maven projects can then apply these tiles declaratively via:

```xml
<tiles>
  <tile>groupId:artifactId:version</tile>
</tiles>
```

## 🔍 Tile vs. Parent POM

| Feature            | Parent POM            | Tile (`tile.xml`)         |
|--------------------|-----------------------|----------------------------|
| Inheritance        | One only              | Multiple tiles supported   |
| Reusability        | Global config         | Plugin-only config         |
| Ideal use case     | Dependency mgmt       | Plugin definitions         |
| IDE support        | Good                  | Limited / workaround needed|

---

## 🏗️ Project Structure

```
maven-tiles/
├── pom.xml         # Maven artifact metadata
└── tile.xml        # Actual tile content
```

### tile.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
  <modelVersion>4.0.0</modelVersion>

  <properties>
    <spotless.version>2.37.0</spotless.version>
  </properties>

  <build>
    <plugins>
      <plugin>
        <groupId>com.diffplug.spotless</groupId>
        <artifactId>spotless-maven-plugin</artifactId>
        <version>${spotless.version}</version>
        <configuration>
          <java>
            <googleJavaFormat />
          </java>
        </configuration>
        <executions>
          <execution>
            <id>spotless-check</id>
            <phase>validate</phase>
            <goals><goal>check</goal></goals>
          </execution>
          <execution>
            <id>spotless-apply</id>
            <phase>process-sources</phase>
            <goals><goal>apply</goal></goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

---

## 🧩 pom.xml for Publishing

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>io.github.y2k6302</groupId>
  <artifactId>maven-tiles</artifactId>
  <version>1.0.6</version>
  <packaging>tile</packaging>

  <name>Spotless Tile</name>
  <description>Maven Tiles for Spotless formatting</description>
  <url>https://github.com/y2k6302/maven-tiles</url>

  <licenses>
    <license>
      <name>Apache License, Version 2.0</name>
      <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
    </license>
  </licenses>

  <developers>
    <developer>
      <id>y2k6302</id>
      <name>Ian Chen</name>
      <email>y2k6302@gmail.com</email>
    </developer>
  </developers>

  <scm>
    <url>https://github.com/y2k6302/maven-tiles</url>
    <connection>scm:git:git://github.com/y2k6302/maven-tiles.git</connection>
    <developerConnection>scm:git:ssh://git@github.com/y2k6302/maven-tiles.git</developerConnection>
  </scm>

  <build>
    <plugins>
      <plugin>
        <groupId>io.repaint.maven</groupId>
        <artifactId>tiles-maven-plugin</artifactId>
        <version>2.37</version>
        <extensions>true</extensions>
      </plugin>
    </plugins>
  </build>

  <profiles>
    <profile>
      <id>release</id>
      <build>
        <plugins>
          <!-- GPG Signature -->
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-gpg-plugin</artifactId>
            <version>3.2.5</version>
            <executions>
              <execution>
                <id>sign-artifacts</id>
                <phase>verify</phase>
                <goals><goal>sign</goal></goals>
              </execution>
            </executions>
          </plugin>

          <!-- Publish to Central -->
          <plugin>
            <groupId>org.sonatype.central</groupId>
            <artifactId>central-publishing-maven-plugin</artifactId>
            <version>0.5.0</version>
            <extensions>true</extensions>
            <configuration>
              <publishingServerId>central</publishingServerId>
              <autoPublish>true</autoPublish>
            </configuration>
          </plugin>

          <!-- Sources & Javadoc -->
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>3.3.1</version>
            <executions>
              <execution>
                <id>attach-sources</id>
                <goals><goal>jar-no-fork</goal></goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>3.6.3</version>
            <executions>
              <execution>
                <id>attach-javadocs</id>
                <goals><goal>jar</goal></goals>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>
</project>
```

---

## 🚀 Deploying to Maven Central

1. Register with [OSSRH](https://central.sonatype.org)
2. Set up `~/.m2/settings.xml` with server credentials and GPG key
3. Deploy:

```bash
mvn clean deploy -P release
```

---

## 🧪 How to Use This Tile in Other Projects

### 1. Apply Tiles Plugin

```xml
<build>
  <plugins>
    <plugin>
      <groupId>io.repaint.maven</groupId>
      <artifactId>tiles-maven-plugin</artifactId>
      <version>2.37</version>
      <extensions>true</extensions>
    </plugin>
  </plugins>
</build>
```

### 2. Import the Tile

```xml
<tiles>
  <tile>io.github.y2k6302:maven-tiles:1.0.6</tile>
</tiles>
```

### 3. (Optional) IDE Support Hack

```xml
<plugin>
  <groupId>com.diffplug.spotless</groupId>
  <artifactId>spotless-maven-plugin</artifactId>
</plugin>
```

This lets IntelliJ or Eclipse recognize the plugin tasks.

---

## ❗ Notes

- Tiles **can only configure `build.plugins`** (not dependencies).
- `tile.xml` will be packaged automatically if:
  - Your `<packaging>` is `tile`
  - You include the `tiles-maven-plugin`
- IDEs won't read tile content unless you help them (see workaround above)

---

## 📚 References

- https://github.com/repaint-io/maven-tiles
- https://central.sonatype.org/publish/publish-guide/
- https://github.com/diffplug/spotless

---

© 2025 Ian Chen (y2k6302) – Licensed under Apache 2.0