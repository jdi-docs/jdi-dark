# JDI Dark Generator

## REST objects generation from Swagger
  You can generate automatically maven project with service object classes, model classes and test classes based on a swagger scheme.

### Algorithm  
 
1. Clone <a href="https://github.com/jdi-testing/jdi-dark-generator" target="_blank">JDI Dark generator</a>  
2. Build JDI Dark generator by command **mvn clean package**  
2. Run generate command for auto-generation.

java -jar DarkGenerator.jar generate [ *command-args* ]

Command may look like this: 
 
   java -jar jdi-dark-rest-generator/target/DarkGenerator.jar generate -i <specification swagger file> -o <output directory> -p <package>
 ``
 
 For example, for <a href="https://petstore.swagger.io/v2/swagger.json" target="_blank">the petshop swagger scheme</a> after running the command  
 
 ``
  java -jar jdi-dark-rest-generator/target/DarkGenerator.jar generate -i https://petstore.swagger.io/v2/swagger.json -o ../sample -p com.petshop  
 ``
 
 the project with the following structure will be created:  
 <img src="images/generated-structure.png" alt="project structure" width="50%" height="50%"/>
 

### Building
After cloning the project, you can build it from the source with this command:

mvn clean package

If you don't have maven installed, you may directly use the included maven wrapper, and build with the command:

./mvnw clean package

### Execution
`java -jar DarkGenerator.jar` *command* [ *command-args* ]

`java -jar DarkGenerator.jar` `generate` - Generate JDI annotated classes from
Open API (Swagger) specification

#### COMMANDS

- `generate`

  Generate JDI annotated classes from Open API/Swagger specification

- `help`

  Display help information


##### GENERATE

###### SYNOPSIS

`java -jar DarkGenerator.jar` `generate` [ { `-a` | `--auth` } *authorization*
] [ `--additional-properties` *additional properties* *...* ] [ `--api-package`
*api package* ] [ `--artifact-id` *artifact id* ] [ `--artifact-version`
*artifact version* ] [ { `-c` | `--config` } *configuration file* ] [
`--codegen-arguments` *codegen arguments* *...* ] [ `-D` *system properties*
*...* ] [ `--date-library` *date library* ] [ `--disable-examples` ] [
`--flatten-inline-schema` ] [ `--git-repo-id` *git repo id* ] [ `--git-user-id`
*git user id* ] [ `--group-id` *group id* ] [ { `-h` | `--help` } ] [
`--hide-generation-timestamp` ] [ `--http-user-agent` *http user agent* ] [ {
`-i` | `--input-spec` } *spec file* ] [ `--ignore-file-override` *ignore file
override location* ] [ `--ignore-import-mapping` *ignore import mapping* ] [
`--import-mappings` *import mappings* *...* ] [ `--instantiation-types`
*instantiation types* *...* ] [ `--language-specific-primitives` *language
specific primitives* *...* ] [ `--model-name-prefix` *model name prefix* ] [
`--model-name-suffix` *model name suffix* ] [ `--model-package` *model package*
] [ { `-o` | `--output` } *output directory* ] [ { `-p` | `--invoker-package` }
*invoker package* ] [ `--release-note` *release note* ] [
`--remove-operation-id-prefix` ] [ `--reserved-words-mappings` *reserved word
mappings* *...* ] [ { `-s` | `--skip-overwrite` } ] [ `--serialization-library`
*serialization library* ] [ `--skip-alias-generation` ] [ { `-t` |
`--template-dir` } *template directory* ] [ `--template-engine` *template
engine* ] [ `--type-mappings` *type mappings* *...* ] [ `--use-oas2` ] [ { `-v`
| `--verbose` } ]

###### OPTIONS

It is obligatory to specify input specification either as command line option or in supplied config file. Other options are not required and have reasonable defaults.

- `-a` *authorization* , `--auth` *authorization*

  adds authorization headers when fetching the swagger definitions remotely.
  Pass in a URL-encoded string of name:header with a comma separating multiple
  values

- `--additional-properties` *additional properties*

  sets additional properties that can be referenced by the mustache templates
  in the format of name=value,name=value. You can also have multiple
  occurrences of this option.

- `--api-package` *api package*

  package for generated api classes

- `--artifact-id` *artifact id*

  artifactId in generated pom.xml

- `--artifact-version` *artifact version*

  artifact version in generated pom.xml

- `-c` *configuration file* , `--config` *configuration file*

  Path to json configuration file. File content should be in a json format
  {"optionKey":"optionValue", "optionKey1":"optionValue1"...} Supported options
  can be different for each language. Run config-help -l {lang} command for
  language specific config options.

- `--codegen-arguments` *codegen arguments*

  codegen arguments

- `-D` *system properties*

  sets specified system properties in the format of name=value,name=value (or
  multiple options, each with name=value)

- `--date-library` *date library*

  Option. Date library to use
  . joda - Joda (for legacy app only)
  . legacy - Legacy java.util.Date (if you really have a good reason not to use
  threetenbp
  . java8-localdatetime - Java 8 using LocalDateTime (for legacy app only)
  . java8 - Java 8 native JSR310 (preferred for jdk 1.8+) - note: this also
  sets "java8" to true
  . threetenbp - Backport of JSR310 (preferred for jdk < 1.8)
  . Default Java 8

- `--disable-examples`

  disable examples

- `--flatten-inline-schema`

  flattenInlineSchema

- `--git-repo-id` *git repo id*

  Git repo ID, e.g. swagger-codegen.

- `--git-user-id` *git user id*

  Git user ID, e.g. swagger-api.

- `--group-id` *group id*

  groupId in generated pom.xml

- `-h` , `--help`

  Display help information

- `--hide-generation-timestamp`

  Hides the generation timestamp when files are generated.

- `--http-user-agent` *http user agent*

  HTTP user agent, e.g. codegen_csharp_api_client, default to
  'Swagger-Codegen/{packageVersion}}/{language}'

- `-i` *spec file* , `--input-spec` *spec file*

  location of the swagger spec, as URL or file (required)

- `--ignore-file-override` *ignore file override location*

  Specifies an override location for the .swagger-codegen-ignore file. Most
  useful on initial generation.

- `--ignore-import-mapping` *ignore import mapping*

  allow generate model classes using names previously listed on import
  mappings.

- `--import-mappings` *import mappings*

  specifies mappings between a given class and the import that should be used
  for that class in the format of type=import,type=import. You can also have
  multiple occurrences of this option.

- `--instantiation-types` *instantiation types*

  sets instantiation type mappings in the format of
  type=instantiatedType,type=instantiatedType.For example (in Java):
  array=ArrayList,map=HashMap. In other words array types will get instantiated
  as ArrayList in generated code. You can also have multiple occurrences of
  this option.

- `--language-specific-primitives` *language specific primitives*

  specifies additional language specific primitive types in the format of
  type1,type2,type3,type3. For example: String,boolean,Boolean,Double. You can
  also have multiple occurrences of this option.

- `--model-name-prefix` *model name prefix*

  Prefix that will be prepended to all model names. Default is the empty
  string.

- `--model-name-suffix` *model name suffix*

  Suffix that will be appended to all model names. Default is the empty string.

- `--model-package` *model package*

  package for generated models

- `-o` *output directory* , `--output` *output directory*

  where to write the generated files. Defaults to generated-code folder

- `-p` *invoker package* , `--invoker-package` *invoker package*

  root package for generated code

- `--release-note` *release note*

  Release note, default to 'Minor update'.

- `--remove-operation-id-prefix`

  Remove prefix of operationId, e.g. config_getId => getId

- `--reserved-words-mappings` *reserved word mappings*

  specifies how a reserved name should be escaped to. Otherwise, the default
  _<name> is used. For example id=identifier. You can also have multiple
  occurrences of this option.

- `-s` , `--skip-overwrite`

  specifies if the existing files should be overwritten during the generation.

- `--serialization-library` *serialization library*

  Serialization library (Default: Jackson)

- `--skip-alias-generation`

  skip code generation for models identified as alias.

- `-t` *template directory* , `--template-dir` *template directory*

  folder containing the template files

- `--template-engine` *template engine*

  template engine

- `--type-mappings` *type mappings*

  sets mappings between swagger spec types and generated code types in the
  format of swaggerType=generatedType,swaggerType=generatedType. For example:
  array=List,map=Map,string=String. You can also have multiple occurrences of
  this option.

- `--use-oas2`

  use OpenAPI v2.0 (Swagger 1.5.x)

- `-v` , `--verbose`

  verbose mode

---

##### HELP

`java -jar DarkGenerator.jar` `help` - Display help information

###### SYNOPSIS

`java -jar DarkGenerator.jar` `help` [ `--` ] [ *command* ]

###### OPTIONS

- `--`

  This option can be used to separate command-line options from the list of
  arguments (useful when arguments might be mistaken for command-line options)

- *command*

                
### Documentation

Documentation can be generated with  running following command:

mvn airline:generate

Documentation will appear in target/help folder.


##SOAP objects generation from WSDL
You can generate automatically JDI Dark SOAP object classes based on your WSDL scheme.

1. Clone <a href="https://github.com/jdi-testing/jdi-dark-generator" target="_blank">JDI Dark generator</a>  
2. Build JDI Dark generator by command **mvn clean package**  
3. Modify pom.xml in jdi-dark-soap-generator: 
    * set wsdlUrls, for example:
    <br />
    ```
      <wsdlUrls>
        <wsdlUrl>https://speller.yandex.net/services/spellservice?WSDL</wsdlUrl>
      </wsdlUrls>
    ```
    * set the package name, for example:
    <br />
    ```
    <packageName>com.epam.jdi.soap.net.yandex.speller.services.spellservice</packageName>
    ```
 
4. Run maven plugin jaxws:wsimport

The project with the following structure is created:  
<img src="images/generated_soap_structure.png" alt="project structure" width="50%" height="50%"/>

**Note:** A few excessive files are autogenerated. Use service objects' files for creating JDI Dark SOAP methods.

