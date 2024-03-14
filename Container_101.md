### 1: Öncelikle sistemdeki tüm container, image ve volumeleri görelim. Bunun için ayrı ayrı listeleme komutlarını girelim. Ve ardından temizlik yapmak adına makinenizdeki tüm containerları, imageleri ve volumeleri temizleyelim.

````
docker ps
docker image ls
docker volume ls
docker container ls


docker volume prune
docker image prune -a
docker container  prune
````

### 2: centos, alpine, nginx, httpd:alpine, ozgurozturknet/adanzyedocker, ozgurozturknet/hello-app, ozgurozturknet/app1 isimli imajları çalıştığımız sisteme çekelim. 

````
docker image pull centos
docker image pull alpine
docker image  pull nginx
docker image pull httpd:alpine
docker image pull ozgurozturknet/adanzyedocker
docker image pull  ozgurozturknet/hello-app
docker image pull  ozgurozturknet/app1

docker image ls
````

### 3: ozgurozturknet/app1 isimli imajdan bir container yaratalım.

````
docker container  run ozgurozturknet/app1 
````

### 4: httpd:alpine isimli imajdan detached bir container yaratalım. Yarattığımız container ismini ve id’sini görelim. 

```
docker container run -d httpd:alpine

docker ps

```

### 5: Yarattığımız bu contaier’ın loglarına bakalım.

```
docker logs (ya ismini ya da id)
```

### 6: Container’ı durduralım, ardından yeniden çalıştıralım ve son olarak container’ı sistemden kaldıralım. 

```
docker ps =>idsine bakarız

docker container stop id
docker container start id 
docker rm -f  id =>container çalıştığı için -f eklememiz lazım
```

### 7 :ozgurozturknet/adanzyedocker isimli imajdan websunucu adında detached ve “-p 80:80” ile portu publish edilmiş bir container yaratalım. Kendi bilgisayarımızın browserından bu web sitesine erişelim.

```
docker container run --name websunucu -d -p 80:80  ozgurozturknet/adanzyedocker 

127.0.0.1
```

### 8: websunucu adlı bu container’ın içerisine bağlanalım. /usr/local/apache2/htdocs klasörünün altına geçelim ve echo “denemedir” >> index.html komutuyla buradaki dosyaya denemedir yazısını ekleyelim. Web tarayıcıya geçerek dosyaya ekleme yapabildiğimizi görmek için refresh edelim. Sonrasında container içerisinden exit ile çıkalım.

```
docker container  exec -it websunucu sh
cd /usr/local/apache2/htdocs
echo “denemedir” >> index.html
exit
```

### 9: websunucu isimli container’ı çalışırken silelim.

```
docker rm -f websunucu

```

### 10: alpine isimli imajdan bir container yaratalım. Ama varsayılan olarak çalışması gereken uygulama yerine “ls” uygulamasının çalışmasını sağlayalım.

```
docker container  run alpine ls
```

### 11: “alistirma1” isimli bir volüme yaratalım. 

```
docker volume  create  alistirma1
docker volume ls
```

### 12: alpine isimli imajdan “birinci” isimli bir container yaratalım. Bu container’ı interactive modda yaratalım ve bağlanabilelim. Aynı zamanda “alistirma1” isimli volume’u bu containerın “/test” isimli folder’ına mount edelim. Bu folder içerisine geçelim ve “touch abc.txt” komutuyla bir dosya yaratalım daha sonra “echo deneme >> abc.txt” komutuyla bu dosyanın içerisine yazı ekleyelim. 

```
docker container run --name birinci  -it -v alistirma1:/test alpine
cd test
touch abc.txt
echo "deneme" >> abc.txt
```

### 13: alpine isimli imajdan “ikinci” isimli bir container yaratalım. Bu container’ı interactive modda yaratalım ve bağlanabilelim. Aynı zamanda “alistirma1” isimli volume’u bu containerın “/test” isimli folder’ına mount edelim. Bu folder içerisinde “ls” komutyla dosyaları listeleyelim ve abc.txt dosyası olduğunu görelim. “cat abc.txt” ile dosyanın içeriğini kontrol edelim. 
```
docker container run --name ikinci -it -v alistirma1:/test alpine
cd test
ls
cat abc.txt

çıktı=>deneme
```

### 14: alpine isimli imajdan “ucuncu” isimli bir container yaratalım. Bu container’ı interactive modda yaratalım ve bağlanabilelim. Aynı zamanda “alistirma1” isimli volume’u bu containerın “/test” isimli folder’ına mount edelim fakat Read Only olarak mount edelim. Bu folder içerisine geçelim ve “touch abc1.txt” komutuyla bir dosya yaratmaya çalışalım. Ve yaratamadığımızı görelim.
```
docker container run --name ucuncu -it -v alistirma1:/test:ro   alpine
cd test
cat abc.txt

touch abc1.txt=>Dosya oluşturamayız.
```

### 15: Bilgisayarımızda bir klasör yaratalım “örneğin c:\deneme” ve bu klasörün içerisinde index.html adlı bir dosya yaratıp bu dosyanın içerisine birkaç yazı ekleyelim.

```
mkdir deneme

powershelle yapıldığında
cd deneme
touch index.html=>dersek hata verir.
New-Item -ItemType File index.html =>deyince index.html adlı bir dosya oluştu.
```

### 16: ozgurozturknet/adanzyedocker isimli imajdan websunucu1 adında detached ve “-p 80:80” ile portu publish edilmiş bir container yaratalım. Bilgisayarımızda yarattığımız klasörü container’ın içerisindeki /usr/local/apache2/htdocs klasörüne mount edelim. Web browser açarak 127.0.0.1’e gidelim ve sitemizi görelim. Daha sonra bilgisayarımızda yarattığımız klasörün içerisindeki index.html dosyasını edit edelim ve yeni yazılar ekleyelim. Web sayfasını refresh ederek bunların geldiğini görelim.

```
docker container run -d --name websunucu1 -p 80:80   -v c:/deneme:/usr/local/apache2/htdocs ozgurozturknet/adanzyedocker
127.0.0.1

```

### 17: Tüm çalışan container’ları silelim. 

```
docker ps -a
docker container prune =>durdurulmuş containerları siler
docker container rm -f (id ya da adını yazarak çalışan containerletı da silebiliriz.)
```
