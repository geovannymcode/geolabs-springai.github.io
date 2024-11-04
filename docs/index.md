# **IA Generativa en Acción: Spring AI con OpenAI, Java 21 y Vaadin**

## **Descripción del Taller**

En este taller interactivo, aprenderás paso a paso cómo trabajar con la IA Generativa desde cero usando Spring AI con OpenAI, Java 21 y Vaadin. En este taller práctico, exploraremos generación de textos, imágenes y audios en un aplicativo frontend con Vaadin. Ideal para quienes buscan dominar Spring AI y Vaadin..

## **Introducción**

## **Requisitos**

- **Java**: Versión 21.
- **Spring Boot**: Versión 3.2.x o superior.
- **Base de Datos**: PostgreSQL.
- **IDE (Entorno de Desarrollo Integrado)**: IntelliJ IDEA, Eclipse, o cualquier otro de tu preferencia.
- **Herramientas de Construcción**: Maven o Gradle.
- **Docker**: Utilizado para la creación y gestión de contenedores, facilitando un entorno de desarrollo consistente y fácil de replicar.

## **Arquitectura Técnica y Servicios Backend**

![GeoLabs SpringAI](./files/SpringAI-Model.png "GeoLabs SpringAI")

### **Dependencias:**

```xml title="pom.xml"
<dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>com.vaadin</groupId>
      <artifactId>vaadin-spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.ai</groupId>
      <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-docker-compose</artifactId>
      <scope>runtime</scope>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.springframework.ai</groupId>
      <artifactId>spring-ai-spring-boot-docker-compose</artifactId>
      <scope>runtime</scope>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  ```