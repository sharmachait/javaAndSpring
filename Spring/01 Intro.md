# Beans
objects whose lifecycles are maintained by the IOC container

# Context
actual app memory where all beans and app specific config are stored

# Spring expression language (SpEL)
Reflection over the dependencies in the IOC container
getting and setting properties of some objects at runtime

# IOC container
manages lifecycles of objects
#### two types of IOC containers
1. BeanFactory - Basic
2. ApplicationContext - advanced
we can add object level middlewares in application context, code that runs when a bean is created or destroyed

# Maven
build tool
allows downloading JARS and adding as project dependencies
Group ID -  multiple projects in the same group
Artifact ID - project name

JARS are down loaded at c://user/chait/.m2/repository