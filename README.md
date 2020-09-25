Solidity Gradle Plugin
======================

Simple Gradle plugin used by the [Web3j plugin](https://github.com/web3j/web3j-gradle-plugin) 
to compile Solidity contracts, but it can be used in any standalone project for this purpose.

## Plugin configuration

To configure the Solidity Gradle Plugin using the plugins DSL or the legacy plugin application, 
check the [plugin page](https://plugins.gradle.org/plugin/org.web3j.solidity). 
The minimum Gradle version to run the plugin is `5.+`.

[//]: # (FIXME Remove these paragraphs after Sokt works without hotkeytlt)

In addition, for the legacy plugin application you will need to add this repository to your `buildscript` configuration:
 
```groovy
buildscript {
    repositories {
        ...
        maven {
            url 'https://dl.bintray.com/hotkeytlt/maven'
        }
    }
    ...
}
```

For the plugins DSL, you will need to add the following configuration in the first line of your `settings.gradle` 
file:

```groovy
pluginManagement {
    repositories {
        gradlePluginPortal()
        maven {
            url 'https://dl.bintray.com/hotkeytlt/maven'
        }
    }
}

Then run this command from your project containing Solidity contracts:

```
./gradlew build
```

After the task execution, the base directory for compiled code (by default 
`$buildDir/resources/solidity`) will contain a directory for each source set 
(by default `main` and `test`), and each of those a directory with the compiled code.


## Code generation

The `solidity` DSL allows to configure the generated code, e.g.:

```groovy
solidity {
    outputComponents = [BIN, ABI, ASM_JSON]
    optimizeRuns = 500
}
```

The properties accepted by the DSL are listed in the following table:

|  Name                      | Type                        | Default value                                     | Description                                                     |
|----------------------------|:---------------------------:|:-------------------------------------------------:|-----------------------------------------------------------------|
| `executable`               | `String`                    | `null` (bundled with the plugin)                  | Solidity compiler path.                                         |
| `version`                  | `String`                    | `0.4.25`                                          | Solidity compiler version.                                      |
| `overwrite`                | `Boolean`                   | `true`                                            | Overwrite existing files.                                       |
| `optimize`                 | `Boolean`                   | `true`                                            | Enable byte code optimizer.                                     |
| `optimizeRuns`             | `Integer`                   | `200`                                             | Set for how many contract runs to optimize.                     |
| `prettyJson`               | `Boolean`                   | `false`                                           | Output JSON in pretty format. Enables the combined JSON output. |
| `ignoreMissing`            | `Boolean`                   | `false`                                           | Ignore missing files.                                           |
| `allowPaths`               | `List<String>`              | `['src/main/solidity', 'src/test/solidity', ...]` | Allow a given path for imports.                                 |
| `evmVersion`               | `EVMVersion`                | `BYZANTIUM`                                       | Select desired EVM version.                                     |
| `outputComponents`         | `OutputComponent[]`         | `[BIN, ABI]`                                      | List of output components to produce.                           |
| `combinedOutputComponents` | `CombinedOutputComponent[]` | `[BIN, BIN_RUNTIME, SRCMAP, SRCMAP_RUNTIME]`      | List of output components in combined JSON output.              |

**Notes:** 
  - Setting the `executable` property will disable the bundled `solc` and use your local or containerized executable:
  ```groovy
    solidity {
        executable = "docker run --rm -v $projectDir:/src satran004/aion-fastvm:latest solc"
        version = '0.4.15'
    }
  ```
  - Use `version` to change the bundled Solidity version. 
    Check the [Solidity releases](https://github.com/ethereum/solidity/releases) 
    for all available versions.
  - `allowPaths` contains all project's Solidity source sets by default.

## Source sets

By default, all `.sol` files in `$projectDir/src/main/solidity` will be processed by the plugin.
To specify and add different source sets, use the `sourceSets` DSL. You can also set your preferred
output directory for compiled code.

```groovy
sourceSets {
    main {
        solidity {
            srcDir {
                "my/custom/path/to/solidity"
             }
             output.resourcesDir = file('out/bin/compiledSol') 
        }
    }
}
```

## Plugin tasks

The [Java Plugin](https://docs.gradle.org/current/userguide/java_plugin.html)
adds tasks to your project build using a naming convention on a per source set basis
(i.e. `compileJava`, `compileTestJava`).

Similarly, the Solidity plugin will add a:

   * `compileSolidity` task for the project `main` source set.
   * `compile<SourceSet>Solidity` for each remaining source set 
     (e.g. `compileTestSolidity` for the `test` source set, etc.). 

To obtain a list and description of all added tasks, run the command:

```
./gradlew tasks --all
```
