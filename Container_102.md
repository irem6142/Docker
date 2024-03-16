### 1: İlk olarak sistemde bir temizlik yapalım ki alıştırmalarımızla çakışma olmasın. Varsa sistemdeki tüm containerları ve kullanıcı tanımlı bridge networkleri silelim.
```
docker container ps -a

docker container prune

docker network ls

docker  network prune ya da docker network rm networkün adı ya da is bilgisi

```

### 2: "alistirma-agi" adında ve 10.10.0.0/16 subnetinde, 10.10.10.0/24 ip-aralığından ip dağıtacak ve gateway olarak da 10.10.10.10 tanımlanacak bir kullanıcı tanımlı bridge network yaratalım. (bu sizin yerel ağınızda kullandığınız bir ağ aralığıysa başka bir cidr kullanabilirsiniz). Bu ağın özelliklerini inspect komutuyla kontrol edin. 

```
docker networ create --driver=bridge --subnet= 10.10.0.0/16 --ip-range=10.10.10.0/24  --gateway=10.10.10.10 alistirma-agi
docker network  inspect alistirma-agi
```
### 3: nginx imajının 1.16 versiyonundan web1 adından detached bir container yaratın. Bu containerı default bridge networküne değil de alistirma-agi networküne bağlı olarak yaratın. Host'un 8080 tcp portuna gelen isteklerin bu containerın 80 portuna gitmesini sağlayın.

```
docker container run -d --name web1 --net alistirma-agi --p 8080:80 nginx:1.16

docker ps
```
### 4: Bu sisteme browser üzerinden erişin ve daha sonra bir kaç kez sayfayı refresh edin. Ardından bu container'ın loglarına erişerek az önceki erişimlerinizin loglarını kontrol edin. 

```
127.0.0.1:8080 ile tarayıcıya erişiriz.
docker logs web1
```

### 5: Container loglarını follow modunda takip ederken browser'dan bu web sitesinde olmayan bir url'e gitmeye çalışın. Örneğin xyz.html Bunun ürettiği hatayı canlı olarak loglardan takip edin. 
```
docker logs -f web1 => -f ile containerlara canlı olarak bağlanıyoruz
Tarayıcıda 127.0.0.1:8080/xyz.html yazarız ve loglardan hatayı alırız.
```
### 6: ozgurozturknet/adanzyedocker imajından test1 adından bir container yaratın. -dit ile yaratın sh shellini açın. Bu container default bridge network'e bağlı olsun. 
```
docker container create  dit  --name test1 -ozgurozturknet/adanzyedocker sh =>dit arka planda çalıştırır.
```
### 7: Bu container'ı "alistirma-agi" networküne de bağlayın.

```
docker network connect alistirma-agi test1
```
### 8: Container'a attach işlemiyle bağlanın ve container içerisinden web1'i pinglemeye çalışın. Container'ı kapamadan çıkın. 

```
docker attach test1
ping web1 =>Aynı ağa bağlı olduğu için pingleyebiliriz.
```
### 9: Bu containerların çalıştığını kontrol edin ve ardından çalışıyor haldeyken bunları silin. 

```
docker ps -a
docker container rm -f  test1
docker container rm  -f web1
```
### 10: Terminalde eğitim klasörünün altındaki kisim4/bolum43 klasörüne geçin. 

```
cd  kisim4/bolum43

```
### 11: ozgurozturknet/webkayit imajından websrv adinda detached bir container yaratın. "alistirma-agi" networküne bağlı olsun. Maksimum 2 logical cpu kullanacak şekilde kısıtlansın. 80 portunu host üstündeki 80 portuyla publish edin. env.list dosyasının bu containera enviroment variable olarak aktarılmasını sağlayın. 

```
docker container run --name websrv -d --net alistirma-agi -p 80:80 --cpus="2" --env-file .\env.list ozgurozturknet/webkayit
dcoker ps -a
```

### 12: ozgurozturknet/webdb imajından mysqldb adinda detached bir container yaratın. "alistirma-agi" networküne bağlı olsun. Maksimum 1gb memory kullanacak şekilde kısıtlansın. envmysql.list dosyasının bu containera enviroment variable olarak aktarılmasını sağlayın. 

```
docker container run --name mysqldb --net alistirma-agi -p 80:80 --memory=1g  --env-file .\envmysql.list  ozgurozturknet/webdb

```
### 13: mysqldb containerının loglarını kontrol ederek düzgün şekilde başlatılabildiğini teyit edin. 

```
docker logs mysqldb
```
### 14: Browser'dan websrv container'ının yayınladığı web sitesine bağlanın. Karşınıza çıkan formu doldurup bir tane jpg dosyayı da seçerek add tuşuna basın. Ardından kayitlari gör diyerek işlemin başarılı olduğunu teyit edin.

```
127.0.0.1 'e giderek işlmleri yapıyoruz.
```
### 15: Oluşturduğunuz containerları ve alistirma-agi'ni silin.

```
docker container prune mysqldb  websrv
docker network rm alistirma-agi
```
