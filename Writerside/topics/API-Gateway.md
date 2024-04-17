# API Gateway

`Spring Cloud Gateway` Spring Cloud tarafından sağlanan bir starter projesidir. Spring Ekosistemi üzerine kurulu bir API 
Gateway sunar, bunlar arasında: Spring, Spring Boot, Spring WebFlux ve Project Reactor bulunmaktadır. Spring Cloud 
Gateway, API'lere yönlendirme yapmayı ve güvenlik, izleme/metricler ve dirençlilik gibi ortak kaygıları bunlara sağlamayı 
amaçlamaktadır.

Spring Cloud Gateway Spring Boot ve Spring WebFlux tarafından sağlanan Netty Server üzerinde çalışır. Geleneksel Servlet
Container'ı (Tomcat gibi) üzerinde çalışmaz. Bununla birlikte bir Spring Boot Gateway applicationunu çalıştırırsanız, 
otomatik olarak Netty Server ile çalışır.

Bir API Gateway, dış bir application/client için birden çok microservice'e tek giriş ve çıkış noktasıdır. Bazen `Ede 
Microservice` olarak da adlandırılır. Dış clientların microservice'lere doğrudan erişimleri kısıtlandığı için, dış 
clientlar ile microserviceler collectionu arasında aracı görevi görür. Gerçek dünya senaryosunda bir dış client şunlar 
olabilir. Masaüstü uygulaması, Mobil Uygulama, Herhangi bir üçüncü taraf uygulama, Dış servisler vb


Bir microservice tabanlı uygulamada, farklı servisler genellikle farklı sunucularda konuşlandırılır. Bu durumda, clientin 
etkileşim için servisin host ve portunu hatırlaması gerekmektedir ki bu çok zahmetlidir. Ayrıca güvenlik ihlali riskleri 
de vardır. Bu nedenle, API Gateway kimlik doğrulamayı onaylar, zekasını kullanır ve client requestini uygun şekilde 
işlemek üzere uygun servise yönlendirir. Güvenlik ve yönlendirmenin yanı sıra, monitoring/metrics ve resiliency'de de 
yardımcı olur.

## API Gateway Avantajları
* Dış clientların her bir microservice'in kimlik doğrulamasından geçmesi gerekmez. Dış clientin başlangıçta sadece bir 
microservicein, yani API Gateway'in kimlik doğrulamasından geçmesi yeterlidir. Eğer kimlik doğrulama API Gateway'de 
reddedilirse, istek başka hiçbir microservice'e devam etmeyecektir. Bu nedenle, dış çağrıların tüm servislerimize 
erişimini sınırlayarak mikroservislerin güvenliğini artırır.

* Microservicelerin endpoint'lerini dış application/client açıklamamıza gerek yoktur.

* Clientler ile microserviceler arasında bir soyutlama sağlar. Client, microservicelerimizin iç mimarisini bilmez. 
Client, microservice instance'larının yerini belirleyemez.

* Tüm client istekleri API Gateway üzerinden yönlendirildiği için, kimlik doğrulama, monitoring/metrics ve resiliency 
gibi cross cutting concerns'leri yalnızca API Gateway'de uygulamamız gerekir. Bu şekilde, geliştirme çabasını en aza 
indirgemeye yardımcı olur.

## API Gateway Dezavantajları
* Her request, güvenlik kontrolünden sonra API Gateway üzerinden geçtiği için performansı yavaşlatabilir.

* Birden fazla microservice için yalnızca bir API Gateway uygularsak ve API Gateway başarısız olursa, request daha fazla 
işlenemez. Bu nedenle, birden fazla Gateway uygulamalı ve trafik load balancer aracılığıyla yönetilmelidir.

* Bazen Gateway'e özgü front-end uygulamamız gerekebilir. Örneğin, üç farklı front-end'imiz var: Android client, iOS 
client, Web Client. Hepsi düzgün çalışabilmesi için bazı özel APIs ve konfigürasyon gerektirebilir. Bu durumu aşmak için 
her tür client için bir tane olmak üzere birden fazla türde service tabanlı mimari oluşturmamız gerekebilir. 
Bazen bu model `Back-end for Front-end(BFF)` olarak adlandırılır.


## API Gateway'e Geçmeden Önce Bilmemiz Gereken Kavramlar

### Route
Route, gateway'in temel yapı taşlarından biridir.

* ID
* Bir destination URI, 
* bir dizi predicate 
* bir dizi filter 

tarafından tanımlanır. 

Bir route, toplam predicate doğru olduğunda eşleşir. Temelde, gelen isteğin yönlendirileceği URL'yi temsil eder.

### PREDICATE
Predicate, Java 8 Function Predicate'tir. Giriş tipi bir Spring Framework ServerWebExchange'dir. Bu, HTTP requestinden, 
örneğin başlıklar veya parametreler gibi herhangi bir şeyle eşleşmenizi sağlar. Kısacası, gelen isteği belirli bir Route 
URL'ye yönlendirmek için uygun olması gereken koşulu içerir.

### FILTER
Bunlar, belirli bir factory ile oluşturulmuş GatewayFilter instance'larıdır. Burada, alt akış isteğini göndermeden önce 
veya sonra istekleri ve yanıtları değiştirebilirsiniz. Basit bir ifadeyle, gelen isteği Route URL'ye göndermeden önce 
veya sonra değiştirmemize olanak tanır


## ROUTING Nedir
Bu, bir Microservice'i predicate ve URL(Path) üzerinden tanımlama ve çalıştırma işlemidir. 

### Static Routing
Eğer bir microservicein tek bir instance'ı varsa (ayrıca mikroservise doğrudan çağrı olarak da bilinir), bu static 
routing olarak adlandırılır. Bu durumda API Gateway isteği doğrudan microservice yönlendirir.

## Dynamic Routing
Bir microservice'in birden fazla instance'ı olduğunda Dynamic Routing devreye girer. Bu durumda, API Gateway, daha az 
yük faktörüne sahip instance'ı bulmak için Eureka'ya ulaşır ve ardından isteği ilgili mikroservise yönlendirir. Load 
balancing için verilen konfigürasyona dayanarak içsel olarak Feign Client kodu üretir.


## PROJE
### Eureka Server
Eureka Server için bir proje oluşturmamız lazım. Daha sonra oluşturacağımız microservicelerin infoları (ayakta olup 
olmadıkları vs..) bu eureka server içerisinde olacak.

Bunun için adımlar şu şekilde olacak.

#### ADIM 1
Bir spring boot projesi başlatıp içerisine aşağıdaki bağımlılıkları ekleyelim.

```VELOCITY
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```VELOCITY
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

#### ADIM 2
`resources` klasörü altındaki application.yml (bütün projede application.properties dosyasını application.yml olarak 
değiştirdim) dosyasına aşağıdaki configleri ekleyelim.

```yaml
server:
  port: 8761

spring:
  application:
    name: EurekaServer

eureka:
  client:
    fetch-registry: false
    register-with-eureka: false
```

> **eureka.client.regşster-with-eureka configi ne işe yarar**
>
> Bu ayar, Eureka Server'ın kendisini Eureka kayıt defterine kaydetmesini önler.
> Yani Eureka Server, kendi Eureka kayıt defterine bir client olarak kaydolmaz.
> Bu genellikle Eureka Server'ın kendisini yönettiği durumlar için kullanılır
> Çünkü bir Eureka Server'ın diğer Eureka Server'lara kaydolması gerekmez.
> Bu yapılandırma, server'ın sadece hizmet kayıt defteri olarak işlev görmesini sağlamak için kullanılır.
>
{style="note"}

> **eureka.client.fetch-registry configi ne işe yarar**
> 
> Bu ayar, Eureka Server'in diger Eureka Server'lardan kayit defterini çekmemesini saglar.
> Bu, genellikle Eureka Server bir stand-alone yapilandirmada calistiginda veya sadece hiz
> ayitlarini almak yerine hizmet kayitlarini saglamak için kullanildiginda tercih edilir.
> Eureka Server'in diger servislerin kayitlarini sürekli olarak güncel tutmasina gerek
> yoktur, çünkü kendisi bir servis saglamaz, sadece kayitlari yönetir.
>
{style="note"}

#### ADIM 3
Main class'ına `@EnableEurekaServer` annotationunu eklememiz lazım.

Yani en sonunda class'ımız şu şekilde olmalı.

```JAVA
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```

### CONFIG SERVER

Proje olarak Config Server oluşturmamız lazım. Bunu oluşturmamızın sebebi, daha sonra oluşturacağımız microservice'lerin
configlerinin config server'dan gelmesini istiyorum.

#### ADIM 1 {id="adim-1_1"}

Bir spring boot projesi oluşturup aşağıdaki bağımlılıkları eklememiz lazım.

```VELOCITY
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```VELOCITY
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```VELOCITY
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### ADIM 2 {id="adim-2_1"}

Bu adımda application.yml dosyasını oluşturacağız.

```yaml
server:
  port: 8181

spring:
  application:
    name: ConfigServer
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/config


eureka:
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    instance-id: ${spring.application.name}:${random.value}
```


> **eureka.client.service-url.defaultZone configi ne işe yarar**
>
> Bu ayar ile eureka server'ın hangi portta ve endpointte ayağa kalkacağını söylüyoruz.
>
{style="note"}

> **eureka.client.instance configi ne işe yarar**
>
> Bu ayar ile eureka server'da config server için özel bir id ile instance veriyoruz.
>
{style="note"}


#### ADIM 3 {id="adim-3_1"}

Bu adımda resources klasörü altında `config` diye bir paket oluşturmamız lazım çünkü yml dosyasında classpath:/config
dosya yolundaki yml'ları araması gerektiğini söyledik.

#### ADIM 4

Main classımızda `@EnableConfigServer` annotationunu eklememiz lazım. Class'ımızın en son hali aşağıdaki gibi olmalı.

```JAVA
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }

}
```

### PAYMENT SERVICE

#### ADIM 1 {id="adim-1_2"}

Aşağıdaki bağımlılıkları eklememiz lazım.

```VELOCITY
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```VELOCITY
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```VELOCITY
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
    <version>4.1.1</version>
</dependency>
```

```VELOCITY
 <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-config</artifactId>
     <version>4.1.0</version>
 </dependency>
```

#### ADIM 2 {id="adim-2_2"}

application.yml dosyamızı oluşturalım.

```yaml
server:
  port: 9090

spring:
  application:
    name: PaymentService
  config:
    import: optional:configserver:http://localhost:8181
  profiles:
    active: dev
  cloud:
    config:
      uri: http://localhost:8181
```

Burada yaptığımız şey, bilgileri nereden çekeceğini verdik.

#### ADIM 3 {id="adim-3_2"}

ConfigServer projesinde resources -> config paketi altına `PaymentService.yml` adında bir dosya oluşturmamız lazım ve 
içerisine aşağıdaki bilgileri yazmamız lazım.

```yaml
project:
  description: "This is Payment Service project description"

eureka:
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    instance-id: ${spring.application.name}:${random.value}
```

#### ADIM 4 {id="adim-4_1"}

controller sınıfını oluşturalım.

```JAVA
@RestController
@RequestMapping("/payment")
public class PaymentController {

    @Value("${project.description}")
    private String description;

    @GetMapping("/project")
    public String getDescription() {
        return description;
    }

    @Value("${server.port}")
    private String port;

    @GetMapping("/info")
    public ResponseEntity<String> showPaymentInfo() {
        return ResponseEntity.ok("FROM PAYMENT SERVICE, Port# is: " + port);
    }
}
```

#### ADIM 5

Main class'ımıza `@EnableDiscoveryClient` annotatinonu eklememiz lazım.

```JAVA
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(PaymentServiceApplication.class, args);
    }

}
```


Buradaki description ve value bilgileri ConfigServer projesi içerisinde resources -> config paketi altında tanımladığımız
PaymentService.yml dosyasından gelecek.

### ORDER SERVICE

#### ADIM 1 {id="adim-1_3"}

Aşağıdaki bağımlılıkları eklememiz lazım.

```VELOCITY
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```VELOCITY
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```VELOCITY
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
    <version>4.1.1</version>
</dependency>
```

```VELOCITY
 <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-config</artifactId>
     <version>4.1.0</version>
 </dependency>
```

#### ADIM 2 {id="adim-2_3"}

application.yml dosyamızı oluşturalım.

```yaml
server:
  port: 6060

spring:
  application:
    name: OrderService
  config:
    import: optional:configserver:http://localhost:8181
  profiles:
    active: dev
  cloud:
    config:
      uri: http://localhost:8181
```

#### ADIM 3 {id="adim-3_3"}

Controller class'ımızı oluşturalım.

```JAVA
@RestController
@RequestMapping("/order")
public class OrderController {

    @Value("${project.description}")
    private String description;

    @GetMapping("/project")
    public String getDescription() {
        return description;
    }

    @Value("${server.port}")
    private String port;

    @GetMapping("/info")
    public ResponseEntity<String> showOrderInfo() {
        return ResponseEntity.ok("FROM ORDER SERVICE, Port# is: " + port);
    }
}
```

#### ADIM 4 {id="adim-4_3"} 

Main class'ımıza `@EnableDiscoveryClient` annotatinonu eklememiz lazım.

```JAVA
@SpringBootApplication
@EnableDiscoveryClient
public class OrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }

}
```


### ÇALIŞTIRMA

* Önce EurekaServer projesini çalıştırmamız lazım. Çünkü sonrasında oluşturduğumuz bütün projeler buraya register olacak.
* İkinci olarak ConfigServer projesini çalıştırmamız lazım. Çünkü diğer service'ler config bilgilerini buradan alacaklar.
* Son adım olarak, sırası fark etmeksizin PaymentService ve OrderService projesini çalıştırmamız lazım. 
* Çalıştırma işlemleri bittiğinde `http://localhost:8761/` endpointine istek attığımız zaman aşağıdaki gibi bir görsel ile
karşılaşağız.

![Create new topic options](EurekaServer_for_API_Gateway.png){ width=1000 }{border-effect=line}

Dikkat ederseniz, yukarıda çalıştırdığımız bütün service'ler register olmuş bir şekilde karşımıza çıktı.

### API Gateway

#### ADIM 1 {id="adim-1_4"}

Spring Boot projesi başlatıp aşağıdaki bağımlılıkları eklememiz lazım.

```VELOCITY
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

```VELOCITY
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### ADIM 2 {id="adim-2_4"}

application.yml file'ı oluşturmamız lazım.

```yaml
server:
  port: 1905

spring:
  application:
    name: ApiGatewayService
```

#### ADIM 3 {id="adim-3_4"}

ConfigServer projesinde resources -> config paketi altında ApiGatewayService.yaml diye bir file açıp içerisini doldurmamız
lazım.

```yaml
project:
  description: "API Gateway project"

eureka:
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    instance-id: ${spring.application.name}:${random.value}
```

#### ADIM 4 {id="adim-4_2"}

Bir config sınıfı oluşturup routing'lerimizi eklememiz lazım.

```
@Configuration
public class ApiGatewayConfiguration {

    @Bean
    public RouteLocator configureRoute(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("paymentId", r -> r.path("/payment/**").uri("http://localhost:9090")) //static routing
                .route("orderId", r -> r.path("/order/**").uri("lb://ORDERSERVICE")) //dynamic routing
                .build();
    }
}
```

#### ADIM 5 {id="adim-5_2"}

Main class'a `@EnableDiscoveryClient` annotationunu eklememiz lazım.

```JAVA
@SpringBootApplication
@EnableDiscoveryClient
public class ApiGatewayServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayServiceApplication.class, args);
    }

}
```

## ROUTING Detaylı İnceleme

```JAVA
@Configuration
public class ApiGatewayConfiguration {

    @Bean
    public RouteLocator configureRoute(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("paymentId", r -> r.path("/payment/**").uri("http://localhost:9090")) //static routing
                .route("orderId", r -> r.path("/order/**").uri("lb://ORDERSERVICE")) //dynamic routing
                .build();
    }
}
```

* <b>`RouteLocator`</b> sınıfı genellikle bir Java `configuration` sınıfı içinde bir `@Bean` olarak tanımlanır. Bu configuration
sınıfı, `RouteLocatorBuilder` aracılığıyla API Gateway'deki rotaları programatik olarak tanımlar.

* <b>`RouteLocatorBuilder:`</b> Rotaları oluşturmak için kullanılan builder modelidir. Bu, bir veya daha fazla rota 
tanımını içeren bir RouteLocator örneği oluşturmak için method chaining kullanır.

* <b>`route() Methodu:`</b> Her bir route() çağrısı, bir Route nesnesi oluşturur. Bu method, rota tanımını başlatır ve bir
tanımlayıcı ve bir lambda ifadesi alır. Lambda ifadesi, `Route.AsyncBuilder` sınıfı tarafından sağlanan yönlendirme 
kurallarını konfigüre eder.

* <b>`path() Metodu:`</b> Bu method, hangi gelen HTTP istek yollarının bu rota ile eşleşeceğini belirler. 
Örneğin, /payment/** patterni /payment ile başlayan tüm URL'leri kapsar.

* <b>`uri() Methodu:`</b> Bu method, eşleşen isteklerin yönlendirileceği hedef URI'yi tanımlar. http://localhost:9090 
gibi statik bir URI veya lb://ORDERSERVICE gibi load balancing kullanarak servis keşif mekanizması üzerinden çözümlenen 
bir URI olabilir.


* <b>`filters() methodu:`</b> Bu method path() methodundan sonra kullanılmalı. İçerisine alabileceği çok fazla parametre
mevcut. İçerisine `Function` parametresi alır. 
<b>addRequestHeader:</b> 2 parametre alır. Birincisi, `headerName`, ikincisi `headerValue`
`.filters(f -> f.addRequestHeader("Hello", "World"))` şeklindeki bir ifadede API Gateway tarafından yönlendirilen 
isteklere "Hello" isimli bir HTTP header'i ekler ve bu başlığın değeri "World" olarak belirlenir.<br />
<b>`addRequestParameter() methodu:`</b> Bu method 2 tane parametre alır.Birincisi `param`, ikincisi `value`değeridir.
Gelen HTTP requestlerine yeni bir query parametresi eklemek için kullanılır. Bu method, isteğin URL'sine belirtilen isim 
ve değerle bir query parametresi ekler. Bu işlem, isteğin hedef servise ulaşmadan önce gerçekleştirilir, böylece hedef 
servis bu eklenen parametreyi istek içinde alabilir ve buna göre işlem yapabilir. Genellikle requestleri modifiye etmek, 
hedef servise ek bilgiler sağlamak veya bazı işlemleri tetiklemek amacıyla yapılır. Örneğin, kullanıcı kimlik doğrulaması 
veya hizmetler arası iletişimde ek veri iletimi gibi durumlar için kullanılabilir.<br />
<b>`cacheRequestBody`:</b> gelen HTTP isteklerinin gövdelerini (bodies) önbelleğe almak için kullanılır. 

```JAVA
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    return builder.routes()
        .route(p -> p
            .path("/modify-request-body")
            .filters(f -> f
                .cacheRequestBody()
                .modifyRequestBody(String.class, String.class,
                    (exchange, s) -> Mono.just(s.toUpperCase()))) // İstek gövdesini büyük harfe çevir
            .uri("http://example.org"))
        .build();
}
```

Bu örnekte /modify-request-body endpointine gelen requestler için istek body'si önce önbelleğe alınır. Sonrasında 
modifyRequestBody() filtresi ile bu body alınır ve tüm karakterleri büyük harfe çevrilir. Bu işlem, önbelleğe alınmış 
body üzerinde yapılır, böylece orijinal akış birden fazla kez okunabilir hale gelir.<br />

<b>`redirect() methodu:`</b> Gelen requestleri belirtilen bir URL'ye yönlendirme işlevi görür. Bu, kullanıcı veya 
istekleri başka bir hedefe otomatik olarak yönlendirmek istediğiniz durumlar için kullanılır. Özellikle, 
eski endpoint'lerin yeni adreslere taşınması, bazı işlemlerin başka servisler tarafından yönetilmesi gerektiğinde ya da 
belirli durumlara özgü yönlendirmeler yapılması gerektiğinde faydalıdır.

