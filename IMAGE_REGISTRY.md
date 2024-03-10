# Docker

### 1.İlk olarak sistemde bir temizlik yapalım ki alıştırmalarımızla çakışma olmasın. Varsa sistemdeki tüm containerları silelim. 
```
docker ps =>Konteynar var mı kontrol ettik.
docker system prune 

```
### 2.Docker logout ve docker login komutlarını kullanarak hesabımızdan logout olup tekrar login olalım. 
```
 docker logout
 docker login
```

### 3. Önceden oluşturduğunuz ve saklamanız gereken imajlar var ise bunları docker hub'a gönderin ve ardından sistemdeki tüm imajları silin
```
docker image ls =>İmajları listeledi.
docker image tag imajınAdi:sürümü  dockerKullanıcıAdı:uygulamaAdı
docker image  push  dockerKullanıcıAdı:uygulamaAdı 
docker image prune -a 

```

### 4.Centos imajının latest ve 7, ubuntu imajının latest, 18.04 ve 20.04, alpine imajının latest, nginx imajının latest ve alpine tagli imajlarını sisteme çekin. 
```
docker pull centos:latest
docker pull centos:7
docker pull ubuntu:latest
docker pull ubuntu:18.04
docker pull ubuntu:20.04
docker pull alpine:latest
docker pull nginx:latest
docker pull nginx:apine
```

### 5.Ubuntu:18.04 imajına dockerhubkullaniciadiniz/alistirma:ubuntu olarak tag ekleyin ve ardından bu yeni imajı docker hub'a gönderin. Alistirma repository'inizden imajı check edin. 
```
docker image tag ubuntu:18.04 dockerhubkullaniciadiniz/alistirma:ubuntu
docker image push dockerhubkullaniciadiniz/alistirma:ubuntu
```
### 6.Bu alistirma.txt dosyasının olduğu klasörde bir Dockerfile oluşturun: 
- Base imaj olarak nginx:latest imajını kullanın
- İmaja LABEL="kendi adınız ve erişim bilgileriniz" şeklinde label ekleyiniz. 
- KULLANICI adında bir enviroment variable tanımlayın ve değer olarak adınızı atayın
- RENK adından bir build ARG tanımlayın
- Sistemi update edin ve ardından curl, htop ve wget uygulamalarını kurun
- /gecici klasörüne geçin ve https://wordpress.org/latest.tar.gz dosyasını buraya ekleyin
- /usr/share/nginx/html klasörüne geçin ve html/${RENK}/ klasörünün içeriğini buraya kopyalayın
- Healtcheck talimatı girelim. curl ile localhost'u kontrol etsin. Başlangıç periodu 5 saniye, deneme aralığı 30s ve
zaman aşımı süresi de 30 saniye olsun. Deneme sayısı 3 olsun. 
- Bu imajdan bir container yaratıldığı zaman ./script.sh dosyasının çalışmasını sağlayan talimatı exec formunda girin. 

```
mkdir Dockerfile

FROM nginx:latest
LABEL maintainer=" irem  @irem4261"
ENV KULLANICI="irem"
ARG RENK
RUN  apt-get update && apt-get install -y  curl,htop,wget
WORKDIR /gecici
ADD  https://wordpress.org/latest.tar.gz .
WORKDIR /usr/share/nginx/html
COPY html/${RENK}/ .
HEALTCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 CMD curl -f http://localhost/ || exit 1
CMD ["./script.sh"]
```
### 7.Bu Dockerfile dosyasından 2 imaj yaratın. Birinci imajda build ARG olarak RENK:kirmizi ikinci imajda da
build ARG olarak RENK:sari kullanın. Kirmizi olan imajın tagi dockerhubkullaniciadiniz/alistirma:kirmizi 
Sari olan imajin tagi dockerhubkullaniciadiniz/alistirma:sari olsun. 
```
docker image build -t irem4261/alistirma:kirmizi --build-arg RENK=kirmizi .

docker image build -t irem4261/alistirma:sari --build-arg RENK=sari  .


docker image ls 

```
### 8.Dockerhubkullaniciadiniz/alistirma:kirmizi imajını kullanarak bir container yaratın. Detach olsun.
Makinenin 80 portuna gelen istekler bu containerın 80 portuna gitsin. Container adi kirmizi olsun.
Browser'dan http://127.0.0.1 sayfasına gidip check edin.  
```
docker container run -d -p 80:80 --name kirmizi irem4261/alistirma:kirmizi

docker ps
```
### 9.Dockerhubkullaniciadiniz/alistirma:sari imajını kullanarak bir container yaratın. Detach olsun.
Makinenin 8080 portuna gelen istekler bu containerın 80 portuna gitsin.
KULLANICI enviroment variable değerini "Deneme" olarak atayın. Container adi sari olsun. 
Browser'dan http://127.0.0.1:8080 sayfasına gidip check edin.
```
docker run container -d -p 8080:80 --name sari --env KULLANICI="deneme" irem4261/alistirma:sari

docker ps
```
### 10.Containerları kapatıp silelim. 
```
docker container stop
docker container rm sari
docker container rm kirmizi

~ docker container rm -f kirmizi sari
```
###  11. Bu alistirma.txt dosyasının olduğu klasörde Dockerfile.multi isimli bir Dockerfile oluşturun: 
- Bu multi stage build alıştırması olacak. 
- Birinci stage'i  mcr.microsoft.com/java/jdk:8-zulu-alpine imajından oluşturun ve stage adı birinci olsun
- /usr/src/uygulama klasörüne geçin ve source klasörünün içeriğini buraya kopyalayın
- "javac uygulama.java" komutunu çalıştırarak uygulamanızı derleyin
- mcr.microsoft.com/java/jre:8-zulu-alpine imajından ikinci aşamayı başlatın. 
- /uygulama klasörüne geçin ve birinci aşamadıki imajın /usr/src/uygulama klasörünün içeriğini buraya kopyalayın
- Bu imajdan container yaratıldığı zaman "java uygulama" komutunun çalışması için talimat girin
```
mkdir Dockerfile.multi

FROM  mcr.microsoft.com/java/jdk:8-zulu-alpine AS birinci 
COPY  /source /usr/src/uygulama 
WORKDIR /usr/src/uygulama
RUN  javac uygulama.java

FROM  mcr.microsoft.com/java/jdk:8-zulu-alpine AS ikinci
WORKDIR /uygulama
COPY --from=birinci /usr/src/uygulama .
CMD ["java","uygulama"]

```

### Bu Dockerfile.multi dosyasından dockerhubkullaniciadiniz/alistirma:java tagli bir imaj yaratın. 
```
docker image build -t  irem4261/alistirma:java -f Dockerfile.multi .
```

### Bu imajdan bir container yaratın ve java uygulamanızın çıkardığı mesajı görün.
```
docker container run  irem4261/alistirma:java

~ docker run  irem4261/alistirma:java
```

###  dockerhubkullaniciadiniz/alistirma:kirmizi, dockerhubkullaniciadiniz/alistirma:sari, dockerhubkullaniciadiniz/alistirma:java
imajlarını Docker hub'a yollayın. 

```
docker  push irem4261/alistirma:kirmizi

docker  push irem4261/alistirma:sari

docker  push irem4261/alistirma:java
```

### Docker hub'daki registry isimli imajdan lokal bir Docker Registry çalıştırın. 
```
docker run -d -p 5000:5000 --restart always --name registry registry
```

### dockerhubkullaniciadiniz/alistirma:kirmizi, dockerhubkullaniciadiniz/alistirma:sari, dockerhubkullaniciadiniz/alistirma:java
imajlarını yeniden tagleyerek bu lokal registry'e gönderin ve ardından bu registry'nin web arayüzünden kontrol edin. 
```
docker iamge tag irem4261/alistirma:kirmizi 127.0.0.1:5000/kirmizi:latest
docker iamge tag irem4261/alistirma:sari 127.0.0.1:5000/sari:latest
docker iamge tag irem4261/alistirma:java 127.0.0.1:5000/java:latest

docker push 127.0.0.1:5000/kirmizi:latest
docker push 127.0.0.1:5000/sari:latest
docker psuh 127.0.0.1:5000/java:latest

Web arayüz kontrol:
127.0.0.1:5000/v2/_catalog

```
