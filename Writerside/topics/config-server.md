# Spring Cloud Config Server

Spring Cloud Config Server, harici yapılandırma (key-value çiftleri veya eşdeğer YAML içeriği) için HTTP kaynak tabanlı 
bir API sağlar.

Spring Cloud Config Server, Spring Cloud projesinin bir parçası olan ve microservice mimarilerinde yapılandırma yönetimini 
merkezi bir şekilde sağlamak için kullanılan bir araçtır.  Bu sunucu, uygulama yapılandırmalarını bir merkezden yönetmemizi 
ve değişiklikleri kolaylıkla uygulamamızı sağlar. Bu sayede, farklı ortamlar için (örneğin, dev, test, prod) yapılandırmaları 
ayrı tutabilir ve ihtiyaç duyulan yapılandırmayı ilgili microservice dinamik olarak sağlayabiliriz.

## Cloud Config Spring Boot Proejsi Adımları 

## LOCAL

### ADIM 1

<p><u>config-service</u> adında bir proje başlatmamız lazım ve bunun içi de öncelikle 
<control>spring-cloud-config-server</control> bağımlılığını eklememiz lazım.</p> 

```velocity
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
```

### ADIM 2

Spring Boot main class'ına 
```velocity
@EnableConfigServer
```
annotationu eklememiz lazım ve son görüntü aşağıdaki gibi olmalı.

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }

}
```

### ADIM 3
Şimdi application.yml dosyasını düzenlemeye geldi.

```yaml
server:
  port: 8585

spring:
  application:
    name: config-server
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/config
```

* <b>spring.profiles.active: native => </b> Bir config-server'da yapılandırma configleri uzak bir sunucudan da gelebilir
(git vs.) local'den de çekilebilir. Bu config, şu an için yapılandırma ayarlarının local'den çekileceğine işaret ediyor.
bir Spring Cloud Config Server uygulamasında, yapılandırma bilgilerinin dosya sistemi üzerinden doğrudan okunacağını belirtir. 
Spring Cloud Config, yapılandırma bilgilerini saklamak ve dağıtmak için çeşitli ortamları destekler; bunlar arasında Git, 
Subversion (SVN), ve dosya sistemi yer alır. native profili, yapılandırma bilgilerinin bir sürüm kontrol sistemi yerine, 
yerel dosya sistemi üzerinden yüklenmesi gerektiğini belirtir. Bu, genellikle geliştirme veya test aşamalarında tercih 
edilir, çünkü bu durumlarda yapılandırmaların hızlı bir şekilde değiştirilip test edilmesi gerekebilir.

native'den başka alabileceği parametreler ise aşağıdaki gibidir.

1. <b>git:</b> Bu, Spring Cloud Config Server'ın en yaygın kullanım senaryolarından biridir. Yapılandırma dosyaları bir 
Git reposunda saklanır. Bu, yapılandırma değişikliklerinin sürüm kontrolü altında tutulmasını ve kolayca yönetilmesini 
sağlar. Git profili, yapılandırmaların merkezi bir yerden yönetilmesi ve dağıtılmasını kolaylaştırır.
2. <b>subversion(svn):</b> Yapılandırma dosyalarını bir Subversion (SVN) reposunda saklamak için kullanılır. Bu, Git'e 
benzer şekilde çalışır ve yapılandırma dosyalarının sürüm kontrolü altında tutulmasını sağlar.
3. <b>valut:</b> HashiCorp Vault, hassas verilerin (örneğin, şifreler, sertifikalar vb.) güvenli bir şekilde saklanması 
ve erişilmesi için kullanılan bir araçtır. vault profili aktifken, Spring Cloud Config Server yapılandırma bilgilerini 
Vault'tan okuyabilir. Bu, özellikle hassas yapılandırma bilgilerinin yönetimi için güçlü bir çözüm sağlar.
4. <b>consul:</b> Consul, HashiCorp tarafından geliştirilen bir hizmet keşif ve yapılandırma aracıdır. consul profili, 
yapılandırma bilgilerinin Consul'dan okunmasını sağlar. Bu, microservice'ler arası iletişim ve yapılandırma bilgilerinin 
dinamik olarak yönetilmesi senaryolarında kullanışlıdır.
5. <b>jdbc:</b> Yapılandırma bilgilerini bir veritabanından okumak için kullanılır. Bu profil, yapılandırma ayarlarını 
SQL tabanlı bir veritabanında saklamanızı sağlar.


* <b>spring.cloud.config.server.native.search-locations:</b> Spring Cloud Config Server'ın native profilini kullanırken, 
yapılandırma dosyalarının fiziksel olarak nerede aranacağını belirtmek için kullanılır.

### ADIM 4
Bu adımda, resources klasörü altına config diye bir folder açmamız gerekmekte. Çünkü application.yml dosyasında search-
location kısmına config değerini verdik.
Bu folder altına da 3 tane yml dosyası açıp içeriklerini doldurabiliriz. Ama burada önemli not şu. dosya isimleri belirli
bir patternde olmalı.

`department-service.yml`, `department-service-dev.yml`, `department-service-test.yml` diyerek 3 farklı yml dosyası 
oluşturabiliriz.

İçeriklerine de sırasıyla

```yaml
project:
  description: "Department service from config-server"
```

```yaml
project:
  description: "Department service from config-server in dev"
```

```yaml
project:
  description: "Department service from config-server in Production"
```

şeklinde doldurabiliriz.

### ADIM 5
Şimdi artık client-config uygulamamızı geliştirebiliriz. Bu uygulama, İlk 4 adımda oluşturduğumuz `config-server` 
uygulamamıza client olacak oradan gerekli dataları çekecek.

Bu adımda department-service adında bir uygulama oluşturup bağımlılık olarak 

```Velocity
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.2.4</version>
</dependency>
```

```Velocity
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>4.1.0</version>
</dependency>
```

bağımlılıklarını ekleyebiliriz.

### ADIM 6
Bu adımda ``application.yml`` dosyamızı oluşturalım.

```yaml
server:
  port: 8181

spring:
  application:
    name: department-service
  config:
    import: optional:configserver:http://localhost:8585
  cloud:
    config:
      uri: http://localhost:8585
  profiles:
    active: dev
```
* <b>spring.config.import:</b> Bu özellik ile config bilgilerinin handi dış kaynaktan alınacağı yeri söylüyoruz.
`optional:configserver:http://localhost:8585` ifadesi, yapılandırma bilgilerinin isteğe bağlı olarak <u>http://localhost:8585</u> 
adresinde çalışan bir Config Server'dan alınabileceğini belirtir. optional kelimesi, Config Server'ın erişilebilir 
olmaması durumunda uygulamanın başlatılmasının engellenmeyeceğini gösterir.

* <b>spring.cloud.config.uri:</b> Config Server'ın URI'sini belirtir. Bu, uygulamanın yapılandırma bilgilerini almak için 
bağlanacağı Config Server'ın adresidir.

> **ÖNEMLİ NOT**
>
> config-server uygulamamızda resources -> config diyerek oluşturduğumuz paketin içerisine 3 tane yml dosyası ekledik.
> Bu yml'lar aynı patternde olmak durumunda. Çünkü department-service içerisinde yazdığımız ``spring.profiles.active = dev``
> yapılandırması direkt olarak config-server içindeki `-dev.yml` dosyasını okuyacak. eğer `spring.profiles.active`kısmına
> test deseydik `-test.yml` dosyasını okuyacaktı. Eğer bu özelliği tamamen silseydik ise en sade olan dosyayı okuyacaktı.
>
{style="note"}


### ADIM 7
Bu adımda bir tane controller class'ı ekliyoruz.

```Java
@RestController
public class HelloController {

    @Value("${project.description}")
    private String projectDescription;

    @GetMapping("/config-test")
    public String getConfigValue() {
        return projectDescription;
    }
```

Artık uygulamamız hazır hale geldi ve test'e geçebiliriz.

Bunun için yapmamız gereken önce config-server projesini sonra da, client-server projesini ayaklandırmak, akabinde
enpointe istek atmak.

## GIT
### ADIM 1 {id="adim-1_2"}
Öncelikle github'da bir repo oluşturup içerisine configlerimizi yazmamız lazım.
aslında içeriği okunacak yaml dosyaları için bir repo açıp yaml'ları yerleştirebiliriz.

### ADIM 2 {id="adim-1_1"}
Bunun için öncelikle client-server uygulamamızda aşağıdaki değişiklikleri yapmamız lazım.
```YAML
server:
  port: 8181

spring:
  profiles:
    active: dev
  application:
    name: department-service
  config:
    import: "optional:configserver:http://localhost:8585"
```

### ADIM 3 {id="adim-3_1"}
Sonrasında config-server'a gidip yaml file'ını düzenlememiz lazım.
```YAML
server:
  port: 8585

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/fsk/spring-boot-cloud-native-config-server
          clone-on-start: true
```

Ve artık test aşamaasına geçebiliriz.
Her iki projeyi ayağa kaldırıp istek attığımız zaman, github'dan veri okunacaktır.
