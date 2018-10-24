The Flink connector library for Pravega supports the Flink Streaming API, Table API, and Batch API, using a common configuration class.  See the below sections for details.

## Table of Contents
- [Common Configuration](#common-configuration)
  - [PravegaConfig Class](#pravegaconfig-class)
  - [Creating PravegaConfig](#creating-pravegaconfig)
  - [Using PravegaConfig](#using-pravegaconfig)
  - [Configuration Elements](#configuration-elements)
  - [Understanding the Default Scope](#understanding-the-default-scope)

## Common Configuration

### PravegaConfig Class
A top-level config object, `PravegaConfig`, is provided to establish a Pravega context for the Flink connector.  The config object automatically configures itself from environment variables, system properties, and program arguments.

Here's a summary of the information sources that `PravegaConfig` uses:

|Setting|Environment Variable /<br/>System Property /<br/>Program Argument|Default Value|
|-------|-------------------------------------------------------------|-------------|
|Controller URI|`PRAVEGA_CONTROLLER_URI`<br/>`pravega.controller.uri`<br/>`--controller`|`tcp://localhost:9090`|
|Default Scope|`PRAVEGA_SCOPE`<br/>`pravega.scope`<br/>`--scope`|-|
|Credentials|-|-|
|Hostname Validation|-|`true`|

### Creating PravegaConfig
The recommended way to create an instance of `PravegaConfig` is to pass an instance of `ParameterTool` to `fromParams`:
```java
ParameterTool params = ParameterTool.fromArgs(args);
PravegaConfig config = PravegaConfig.fromParams(params);
```

If your application doesn't use the `ParameterTool` class that is provided by Flink, create the `PravegaConfig` using `fromDefaults`:
```java
PravegaConfig config = PravegaConfig.fromDefaults();
```

The `PravegaConfig` class provides a builder-style API to override the default configuration settings:
```java
PravegaConfig config = PravegaConfig.fromDefaults()
    .withControllerURI("tcp://...")
    .withDefaultScope("SCOPE-NAME")
    .withCredentials(credentials)
    .withHostnameValidation(false);
```

### Using PravegaConfig
All of the various source and sink classes provided with the connector library have a builder-style API which accepts a `PravegaConfig` for common configuration.  Pass a `PravegaConfig` object to the respective builder via `withPravegaConfig`.  For example:
```java
PravegaConfig config = ...;

FlinkPravegaReader<MyClass> pravegaSource = FlinkPravegaReader.<MyClass>builder()
    .forStream(...)
    .withPravegaConfig(config)
    .build();
```

### Understanding the Default Scope
Pravega organizes streams into _scopes_ for the purposes of manageability.  The `PravegaConfig` establishes a default scope name that is used in two scenarios:
1. For resolving unqualified stream names when constructing a source or sink.  The sources and sinks accept stream names that may be _qualified_ (e.g. `my-scope/my-stream`) or _unqualified_ (e.g. `my-stream`).
2. For establishing the scope name for the coordination stream underlying a Pravega reader group.

Note that the `FlinkPravegaReader` and the `FlinkPravegaTableSource` use the default scope configured on `PravegaConfig` as their reader group scope, and provide `withReaderGroupScope` as an override.  The scope name of input stream(s) doesn't influence the reader group scope.

