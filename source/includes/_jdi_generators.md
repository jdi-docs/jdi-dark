#JDI Dark Generators
##Generation from Swagger
  You can auto-generate maven project with service object classes, model classes and also test classes based on a swagger scheme.
  
1. Clone <a href="https://github.com/jdi-testing/jdi-dark-generator" target="_blank">Jdi dark generator</a>  
2. Build Jdi dark generator by command **mvn clean package** (If you don't have maven installed, you may directly use the included maven wrapper, and build with the command: *./mvnw clean package*)  
2. Run this command for auto-generation:  
 
  **java -jar jdi-dark-rest-generator/target/DarkGenerator.jar generate -i \<specification swagger file\> -o \<output directory\> -p \<package\>**
  
For example, for <a href="http://petstore.swagger.io/v2/swagger.json" target="_blank">the petshop swagger scheme</a> after running the command  

*java -jar jdi-dark-rest-generator/target/DarkGenerator.jar generate -i http://petstore.swagger.io/v2/swagger.json  -o ../sample -p com.petshop*  
  
will be created the project with structure as on this image:  
<img src="images/generated-structure.png" alt="project structure" width="50%" height="50%"/>

**Options**

|Method | Description 
--- | --- 
**-i \<spec file\> or --input-spec \<spec file\>** | location file specification
**-o \<output directory\> or --output \<output directory\>** |  project directory name
**--invoker-package \<invoker package\> or -p\<invoker package\>** | name main package 
**--model-package \<model package\>** |  name package for model classes
**--api-package \<api package\>** |  name package for api classes
**-a \<authorization\> or --auth \<authorization\>** |  add auth data in header(specify in key:value format)
**--artifact-id \<artifact id\>** | project artifact id
**--artifact-version \<artifact version\>** | project artifact version
**--group-id \<group id\>** | project group id
