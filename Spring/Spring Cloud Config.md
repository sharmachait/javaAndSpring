Benefit 1 - can update the values without requiring restart of applications
Benefit 2 - supports encryption
Benefit 2 - Central hub for configuration management
All our applications can subscribe to the centralized config server as clients and get configurations from it during startup
we can configure a data store to store all the configurations, like a database, or a git repo
the server loads the configuration properties and sees the distribution of the configuration to multiple applications

https://github.com/sharmachait/spring-boot-microservices/pull/1/files#diff-877a777c81c2ec05a413f99fa0e9c3727cfc6e2ebb7586a3cc9304b490b305d5