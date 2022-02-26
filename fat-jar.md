# Packaging s2-app as a fat Jar

## What is a fat-jar ?

An uber-jar also known as a "fat-jar", is a level up from a simple JAR, defined as one that contains both your package and all its dependencies in one single JAR file.

The advantage is that you can distribute your uber-jar and not care at all whether or not dependencies are installed at the destination, as your uber-jar actually has no dependencies.

### **There are two common methods for constructing an uber jar:**

* **Unshaded**: Unpack all JAR files, then repack them into a single JAR. Works with Java's default class loader. Tools [maven-assembly-plugin](https://maven.apache.org/plugins/maven-assembly-plugin/)
* **Shaded**: Same as unshaded, but rename (i.e., "shade") all packages of all dependencies. Works with Java's default class loader. Avoids some (not all) dependency version clashes. Tools [maven-shade-plugin](https://maven.apache.org/plugins/maven-shade-plugin/)


We will investigate both the options further.

## Maven-Assembly-Plugin
----------------------

link: [maven-assembly-plugin](https://maven.apache.org/plugins/maven-assembly-plugin/)

The Assembly Plugin for Maven enables developers to combine project output into a single distributable archive that also contains dependencies, modules, site documentation, and other files.

### Usage 
To bind the single goal to a project's build lifecycle, you can add this configuration(`pom.xml`) (assuming you're using the `jar-with-dependencies` prefabricated descriptor):

```
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.3.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <configuration>
                            <descriptorRefs>
                                <descriptorRef>jar-with-dependencies</descriptorRef>
                            </descriptorRefs>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

Then, to create a project assembly, simple execute the normal package phase from the default lifecycle:

```
mvn package
```

The resultant jar will be there in the target directory.

<br/>

## Maven-Shade-Plugin
-----------

link: [maven-shade-plugin](https://maven.apache.org/plugins/maven-shade-plugin/)

This plugin provides the capability to package the artifact in an uber-jar, including its dependencies and to shade - i.e. rename - the packages of some of the dependencies.

### Usage
To Configure the plugin to the project into the project, we must add the below code to our `pom.xml` file.

```
<build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.2.4</version>
        
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```

The goals for the Shade Plugin are bound to the package phase in the build lifecycle.
```
mvn package
```

Note: It can change the jar and cause regression it the software.

