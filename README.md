# Microservices Exam Project

Simple microservices architecture using Spring Boot and Docker. Projet Examen Microservices & DevOps
Ce projet est une architecture microservices complète avec communication synchrone (REST) et asynchrone (Message Broker).

Architecture
service client (Port 8081) : Gestion des clients.
product-service (Port 8082) : Gestion des produits et du stock.
service-order (Port 8083) : Orchestration des commandes.
Technologies
Java 17 & Spring Boot 3.2
Spring Cloud OpenFeign (Synchronisation de communication)
Spring AMQP / RabbitMQ (Communication asynchrone)
Docker & Docker Compose
Base de données H2 (Base de données en mémoire)
Actions GitHub (Pipeline CI/CD)
Prérequis
Docker Desktop installé et lancé.
(Optionnel) Java 17 et Maven pour lancer en local sans Docker.
Lancement du projet
Ouvrir un terminal à la racine du projet.
Lancer la commande :
docker-compose up --build
Attendre que tous les services soient démarrés (logs "Started ... Application »).
Tests des API (Scénario)
Utiliser Postman ou cURL.

1. Créer un Client
POST : « jsonhttp://localhost:8081/api/customers
{
    "name": "Professeur",
    "email": "prof@ecole.com"
}
Notez l’ID retourné (ex : 1).

2. Créer un Produit
POST http://localhost:8082/api/products
{
    "name": "Ordinateur",
    "price": 1200.0,
    "quantity": 10
}
Notez l’ID retourné (ex : 1).

3. Passer une Commande (Test Synchrone + Asynchrone)
POST http://localhost:8083/api/orders?cid=1&pid=1&qty=2

Ce qui se passe :

Synchrone : Order-Service appelle Customer-Service et Product-Service pour vérifier l’existence.
Asynchrone : Order-Service envoie un message à RabbitMQ. Product-Service reçoit le message et met à jour le stock.
4. Vérifier la mise à jour du jour

ALLEZ http://http://localhost:8082/api/products/1

Tests API
Créer Client (POST http://localhost:8081/api/customers)
Corps : {"name": "Test", "email": "test@test.com"}
Créer Produit (POST http://localhost:8082/api/products)
Corps : {"name": "Laptop", "price": 999.0, "quantity": 10}
Créer Commande (POST http://localhost:8083/api/orders?cid=1&pid=1&qty=2)
Vérifie les logs pour voir la communication.
Vérifier Stock (GET http://localhost:8082/api/products/1)
Le stock doit avoir diminué de 2 (Async).
""")
---------------------------------------------------------
1. SERVICE CLIENT
---------------------------------------------------------
create_file(« service client/pom.xml », « "<projet xmlns="http://maven.apache.org/POM/4.0.0 »
xsi :schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd »
xmlns :xsi="http://www.w3.org/2001/XMLSchema-instance"> <modelVersion>4.0.0</modelVersion> <groupId>com.examen</groupId>
<artefactId>service client</artefactId>
<version>1.0.0</version >
<parent> <groupId>org.springframework.boot</groupId> <artefactId>spring-boot-starter-parent</artifactId> <version>3.2.0</version> </parent> <properties><java.version>17</java.version></properties> <dependencies> <dependency><groupId>org.springframework.boot</groupId><artefactId>spring-boot-starter-web</artifactId></dependency> <dependency><groupId>org.springframework.boot</ groupId><artefactId>spring-boot-starter-data-jpa</artifactId></dependency> <dependency><groupId>com.h2database</groupId><artifact<artifactId>h2</artifactId><scope>runtime</scope></dependency> <dependency><groupId>org.projectlombok</groupId><artefactId>lombok</artifactId><optional>true</optional></dependency> </dependencies> </project>""")

create_file(« customer-service/Dockerfile », « "FROM maven :3.9.5-eclipse-temurin-17 AS build
WORKDIR /app
BIEN pom.xml.
BIEN reçu ./src
RUN mvn clean package -DskipTests
DE THE eclipse-temurin : 17-jre-jammy
WORKDIR /app
COPIER --from=build /app/target/customer-service-1.0.0.jar app.jar
EXPOSE 8080
ENTRÉE [« java », « -jar », « app.jar"] » « »)

create_file (« customer-service/src/main/java/com/examen/customer/CustomerServiceApplication.java », « "package com.examen.customer ;
import org.springframework.boot.SpringApplication ;
import org.springframework.boot.autoconfigure.SpringBootApplication ;
@SpringBootApplication
classe publique CustomerServiceApplication {
public static void main(String[] args) { SpringApplication.run(CustomerServiceApplication.class, args) ; }
}""")

create_file(« customer-service/src/main/java/com/examen/customer/model/Customer.java », « "package com.examen.customer.model ;
importez Jakarta.Persistence.* ;
Importez Lombok. Données ;
@Data
@Entity
Public Class Customer {
@Id @GeneratedValue(stratégie = TypeGénération.IDENTIDAD)
le privé Long id ;
nom privé String ;
email privé String ;
}""")

create_file(« customer-service/src/main/java/com/examen/customer/repository/CustomerRepository.java », « "package com.examen.customer.repository ;
import com.examen.customer.model.Customer ;
importe : org.springframework.data.jpa.repository.JpaRepository ;
interface publique CustomerRepository étend JpaRepository<Customer, Long> { }"" »)

create_file (« customer-service/src/main/java/com/examen/customer/controller/CustomerController.java », « "package com.examen.customer.controller ;
import com.examen.customer.model.Customer ;
importation com.examen.customer.repository.CustomerRepository ;
import org.springframework.beans.factory.annotation.Autowired ;
import org.springframework.web.bind.annotation.* ;
importer java.util.List ;
@RestController
@RequestMapping (« /api/clients »)
classe publique CustomerController {
@Autowired dépôt privé CustomerRepository ;
@GetMapping Liste publique<Customer> getAll() { return repo.findAll() ; }
@PostMapping public Customer create(@RequestBody Customer c) { return repo.save(c) ; }
@GetMapping(« /{id} ») public Customer getById(@PathVariable Long id) { return repo.findById(id).orElse(null) ; }
}""")

create_file(« customer-service/src/main/resources/application.properties », « "server.port=8081
spring.application.nom=service client
Spring.DataSource.URL=jdbC :H2 :Mem :CustomerDB
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=mot de passe
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true"" »)

---------------------------------------------------------
2. SERVICE PRODUIT
---------------------------------------------------------
create_file(« produit-service/pom.xml », « " »<projet xmlns="http://maven.apache.org/POM/4.0.0 »
xsi :schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd »
xmlns :xsi="http://www.w3.org/2001/XMLSchema-instance">
<modèleVersion>4.</modelVersion> <groupId>com.examen</groupId>
<artefactId>product-service</artifactId> <version>1</version> <parent> <groupId>org</groupId> <artefactId>spring</artifactId>
<version>3.2.0</version >
</parent> <propriétés><java.version>17</propriétés> <dépendances> <dépendance><groupId>org.springframework.boot</groupId><artefactId>spring-boot</artifactId></dependency> <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-st</artifactId></dependency> <dependency><groupId>com.h2database</groupId><artifactId>h2</artifactId><scope>runtime</scope></dependency> <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-am</artifactId></dependency> <dependency><groupId>org.projectl</groupId><artefactId>lombok</artifactId><optional>true</optional></dependency> </dependencies</dépendances> </projet>""")

create_file(« product-service/Dockerfile », « "FROM maven :3.9.5-eclipse-temurin-17 AS build
WORKDIR /app
BIEN pom.xml.
BIEN reçu ./src
RUN mvn clean package -DskipTests
DE THE eclipse-temurin : 17-jre-jammy
WORKDIR /app
COPY --from=build /app/target/product-service-1.0.0.jar app.jar
EXPOSE 8080
ENTRÉE [« java », « -jar », « app.jar"] » « »)

create_file(« product-service/src/main/java/com/examen/product/ProductServiceApplication.java », « "package com.examen.product ;
import org.springframework.boot.SpringApplication ;
import org.springframework.boot.autoconfigure.SpringBootApplication ;
@SpringBootApplication
classe publique ProductServiceApplication {
public static void main(String[] args) { SpringApplication.run(ProductServiceApplication.class, args) ; }
}""")

create_file(« product-service/src/main/java/com/examen/product/model/Product.java », « "package com.examen.product.model ;
importez Jakarta.Persistence.* ;
Importez Lombok. Données ;
@Data
@Entity
classe publique Product {
@Id @GeneratedValue(stratégie = TypeGénération.IDENTIDAD)
le privé Long id ;
nom privé String ;
privé double prix ;
quantité entière privée ;
}""")

create_file (« product-service/src/main/java/com/examen/product/repository/ProductRepository.java », « "package com.examen.product.repository ;
import com.examen.product.model.Product ;
importe : org.springframework.data.jpa.repository.JpaRepository ;
interface publique ProductRepository étend JpaRepository<Product, Long> { }"" »)

create_file(« product-service/src/main/java/com/examen/product/controller/ProductController.java », « "package com.examen.product.controller ;
import com.examen.product.model.Product ;
importation com.examen.product.repository.ProductRepository ;
import org.springframework.beans.factory.annotation.Autowired ;
import org.springframework.web.bind.annotation.* ;
importer java.util.List ;
@RestController
@RequestMapping (« /api/products »)
classe publique ProductController {
@Autowired dépôt privé ProductRepository ;
@GetMapping Liste publique<Product> getAll() { return repo.findAll() ; }
@PostMapping public Product create(@RequestBody Product p) { return repo.save(p) ; }
@GetMapping(« /{id} ») public Product getById(@PathVariable Long id) { return repo.findById(id).orElse(null) ; }
}""")

create_file (« product-service/src/main/java/com/examen/product/consumer/ProductConsumer.java », « "package com.examen.product.consumer ;
import com.examen.product.model.Product ;
importation com.examen.product.repository.ProductRepository ;
import org.springframework.amqp.rabbit.annotation.RabbitListener ;
import org.springframework.beans.factory.annotation.Autowired ;
import org.springframework.stereotype.Service ;
importer java.util.Map ;

@Service
classe publique ProductConsumer {
@Autowired dépôt privé ProductRepository ;

@RabbitListener(files = « order.created.queue »)
public void handleOrderCreated(Map<String, Object> message) {
Long productId = Long.valueOf(message.get(« productId »).toString()) ;
Entière qty = (Entier) message.get(« quantité ») ;

Product p = repo.findId(productId).orElse(null) ;
if(p != null) {
p.setQuantity(p.getQuantity() - quantité) ;
repo.save(p) ;
System.out.println (« ASYNC : Stock mis à jour pour produit " + productId) ;
}
}
}""")

create_file(« product-service/src/main/resources/application.properties », « "server.port=8082
spring.application.name=product-service
Spring.DataSource.URL=jdbC :H2 :MeM :ProductDB
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=mot de passe
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.rabbitmq.host=rabbitmq"" »)

---------------------------------------------------------
3. ORDRE DE SERVICE
---------------------------------------------------------
create_file(« service-ordre/pom.xml », «  »<projet xmlns="http://maven.apache.org/POM/4.0.0 »
xsi :schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd »
xmlns :xsi="http://www.w3.org/2001/XMLSchema-instance">
<modèleVersion>4</modelVersion>
<groupId>com.examen</groupId> <artefactId>order-service</artifactId> <version>1.0.0</version> <parent> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-parent</artifactId> <version>3.2</version >
</parent> <properties><java.version>17</java.version></properties> <dependencies> <dependency><groupId>org.springframework.boot</groupId><artefactId>spring-boot-starter-web</artifactId></dependency> <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-data-jpa</artifactId></dependency>
<dependency><groupId>com.h2database</groupId><artefactId>h2</artifactId><périmètre > durée d’exécution</portée ></dépendance> <dépendance<dependency><groupId>org.springframework.boot</groupId><artefactId>spring-boot-starter-amqp</artifactId></dépendance> <dépendance<dependency><groupId>org.springframework.cloud</groupId><artefactId>spring-cloud-starter</artefactId></dépendance>
<dependency><groupId>org.projectlombok</groupId><artefactId>lombok</artifactId><optional>true</optional></dependency> </dependencies> <dependencyManagement>
<dépendances> <dépendance> <groupId>org.springframework.cloud</groupId> <artefactId>spring-cloud-dependencies</artifactId> <version>2023.0.0</version>
<type>pom</type> <scope>import</champ de poids> </dépendances> </dépendances> </gestion des dépendances> </projet>""")

create_file(« order-service/Dockerfile », « "FROM maven :3.9.5-eclipse-temurin-17 AS build
WORKDIR /app
BIEN pom.xml.
BIEN reçu ./src
RUN mvn clean package -DskipTests
DE THE eclipse-temurin : 17-jre-jammy
WORKDIR /app
COPY --from=build /app/target/order-service-1.0.0.jar app.jar
EXPOSE 8080
ENTRÉE [« java », « -jar », « app.jar"] » « »)

create_file (« order-service/src/main/java/com/examen/order/OrderServiceApplication.java », « "package com.examen.order ;
import org.springframework.boot.SpringApplication ;
import org.springframework.boot.autoconfigure.SpringBootApplication ;
import org.springframework.cloud.openfeign.EnableFeignClients ;
@SpringBootApplication
@EnableFeignClients
classe publique OrderServiceApplication {
public static void main(String[] args) { SpringApplication.run(OrderServiceApplication.class, args) ; }
}""")

create_file(« order-service/src/main/java/com/examen/order/model/Order.java », « "package com.examen.order.model ;
importez Jakarta.Persistence.* ;
Importez Lombok. Données ;
@Data
@Entity
classe publique Ordre {
@Id @GeneratedValue(stratégie = TypeGénération.IDENTIDAD)
le privé Long id ;
privé Long customerId ;
privé Long productId ;
quantité entière privée ;
}""")

create_file(« order-service/src/main/java/com/examen/order/repository/OrderRepository.java », « "paquet com.examen.order.repository ;
import com.examen.order.model.Order ;
importe : org.springframework.data.jpa.repository.JpaRepository ;
interface publique OrderRepository étend JpaRepository<Order, Long> { }"" »)

create_file(« order-service/src/main/java/com/examen/order/dto/CustomerDTO.java », « "package com.examen.order.dto ;
Importez Lombok. Données ;
@Data
classe publique CustomerDTO { private Long id ; private Nom de la chaîne ; }"" »)

create_file(« order-service/src/main/java/com/examen/order/dto/ProductDTO.java », « "package com.examen.order.dto ;
Importez Lombok. Données ;
@Data
classe publique ProductDTO { privé Long id ; privé Nom de chaîne ; privé Double prix ; }"" »)

create_file(« order-service/src/main/java/com/examen/order/client/CustomerClient.java », « "package com.examen.order.client ;
import com.examen.order.dto.CustomerDTO ;
import org.springframework.cloud.openfeign.FeignClient ;
import org.springframework.web.bind.annotation.GetMapping ;
import org.springframework.web.bind.annotation.PathVariable ;
@FeignClient(name = « customer-service », url = « ${customer.service.url} »)
interface publique CustomerClient {
@GetMapping(« /api/customers/{id} »)
CustomerDTO getCustomer(@PathVariable (« id ») Long id) ;
}""")

create_file(« order-service/src/main/java/com/examen/order/client/ProductClient.java », « "package com.examen.order.client ;
import com.examen.order.dto.ProductDTO ;
import org.springframework.cloud.openfeign.FeignClient ;
import org.springframework.web.bind.annotation.GetMapping ;
import org.springframework.web.bind.annotation.PathVariable ;
@FeignClient(name = « product-service », url = « ${product.service.url} »)
interface publique ProductClient {
@GetMapping(« /api/products/{id} »)
ProductDTO getProduct(@PathVariable (« id ») Long id) ;
}""")

create_file (« order-service/src/main/java/com/examen/order/service/OrderService.java », « "package com.examen.order.service ;
import com.examen.order.client.CustomerClient ;
import com.examen.order.client.ProductClient ;
import com.examen.order.model.Order ;
importation com.examen.order.repository.OrderRepository ;
import org.springframework.amqp.rabbit.core.RabbitTemplate ;
import org.springframework.beans.factory.annotation.Autowired ;
import org.springframework.stereotype.Service ;
importer java.util.HashMap ;
importer java.util.Map ;

@Service
classe publique OrderService {
@Autowired dépôt privé OrderRepository ;
@Autowired privé ClientClientClientClient ;
@Autowired privé ProduitClient produit ;
@Autowired RabbitTemplate privé rabbitTemplate ;

public order placeOrder(Long customerId, Long productId, Integer qty) {
// 1. Synchrone : Vérifier Client
if(customerClient.getCustomer(customerId) == null)
lancer un nouveau RuntimeException (« Client inconnu » ») ;

// 2. Synchrone : Vérifier Produit
if(productClient.getProduct(productId) == null)
lancer un nouveau RuntimeException (« Produit inconnu ») ;

// 3. Sauvegarde Commandement
Ordre o = nouvel ordre() ;
o.setCustomerId(customerId) ;
o.setProductId(productId) ;
o.setQuantity(qty) ;
repo.save(o) ;

// 4. Asynchrone : Notification Stock
Map<String, Object> event = new HashMap<>() ;
event.put (« productId », productId) ;
event.put (« quantité », qty) ;
rabbitTemplate.convertAndSend (« order.created.queue », événement) ;

retour o ;
}@PostMapping
public order create(@RequestParam long cid, @RequestParam long pid, @RequestParam integer qty) {
return service.placeOrder(cid, pid, qty) ;
}
}""")

create_file(« order-service/src/main/java/com/examen/order/controller/OrderController.java », « "package com.examen.order.controller ;
import com.examen.order.model.Order ;
import com.examen.order.service.OrderService ;
import org.springframework.beans.factory.annotation.Autowired ;
import org.springframework.web.bind.annotation.* ;
@RestController
@RequestMapping (« /api/orders »)
classe publique OrderController {
@Autowired service privé OrderService ;

}""")

create_file(« order-service/src/main/resources/application.properties », « ""server.port=8083
spring.application.name=order-service
Spring.DataSource.URL=jdbC :H2 :MeM :OrderDB
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=mot de passe
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.rabbitmq.host=rabbitmq

URLs Docker pour communication Synchrone (Feign)
customer.service.url=http://customer-service:8080
product.service.url=http://product-service:8080""")

---------------------------------------------------------
4. DEVOPS
---------------------------------------------------------
create_file(« docker-compose.yml », « ""version : '3.8'
Services :
rabbitmq :
Image : rabbitmq :3-management
Ports :
- "5672:5672"
- "15672:15672"

Service client :
build : ./customer-service
Ports : [« 8081:8080 »]

Service produit :
build : ./product-service
Ports : [« 8082:8080 »]
depends_on : [rabbitmq]
environnement : [SPRING_RABBITMQ_HOST=rabbitmq]

Commande de service :
build : ./order-service
Ports : [« 8083:8080 »]
depends_on : [service client, service produit, rabbitmq]
Environnement :
- SPRING_RABBITMQ_HOST=rabbitmq
- CUSTOMER_SERVICE_URL=http://customer-service:8080
- PRODUCT_SERVICE_URL=http://product-service:8080
""")

create_file(« .github/workflows/main.yml », « ""name : CI/CD Pipeline
Sur :
Push :
Branches : [ Principal ]
Emplois :
Construire et pousser :
Fonctionne-CONTINU : Ubuntu-Latest
Stratégie :
Matrice :
Service : [Service client, Produit-Service, Commande-Service]
Étapes :
- utilisations : actions/checkout@v3
- utilisations : actions/configuration-java@v3
avec : { Java-Version : '17', Distribution : 'Temurin' }
- nom : Build Maven
run : mvn -f {{ secrets. DOCKER_USERNAME }}
mot de passe : {{ matrix.service }}
Push : Vrai
Tags : {{ Matrix.service }} :latest
""")
matrix.Service/pom.xm lc leanpackage−DskipTests−USsays :docker/l ogin−action@v2with :user name :
Secrets.DOCKER 
P
​
 UNSS,W OUD−USsays :docker/build−push−act ion@v4with :con tex t :./
Secrets.DOCKER 
U
​
 SERNAME/

La quantité doit être passée de 10 à 8 automatiquement.

Suivi CI/CD
Le pipeline GitHub Actions ('.github/workflows/main.github/workflows/main.yml) s’exécute à chaque push sur la branche main. Il build les 3 services et pousse les images sur DockerHub.
