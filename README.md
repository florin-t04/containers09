# Lucrarea de laborator 09: Optimizarea imaginilor Docker

## Scopul lucrării
Scopul lucrării este de a se familiariza cu metodele de optimizare a imaginilor.


## Sarcina
Compararea diferitelor metode de optimizare a imaginilor:
- Ștergerea fișierelor temporare și a dependențelor neutilizate
- Reducerea numărului de straturi
- Utilizarea unei imagini de bază minime
- Reambalarea imaginii
- Utilizarea tuturor metodelor combinate

## Execuție

### Pregătirea mediului
- A fost instalat Docker pe calculator.
- A fost creat un repository `containers09`.
- În directorul `containers09` a fost creat un subdirector `site` cu fișierele html, css și js ale site-ului.

### Crearea imaginii inițiale 
Am creat un `Dockerfile.raw` cu următorul conținut:

```dockerfile
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y

# install nginx
RUN apt-get install -y nginx

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```
Am construit imaginea cu numele mynginx:raw:
```bash
docker image build -t mynginx:raw -f Dockerfile.raw .
```

### Eliminarea dependențelor neutilizate și a fișierelor temporare
Am creat un `Dockerfile.clean` cu următorul conținut:
```dockerfile
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y

# install nginx
RUN apt-get install -y nginx

# remove apt cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```
Am asamblat  imaginea cu numele mynginx:clean și am verificat dimensiunea:
```bash
docker image build -t mynginx:clean -f Dockerfile.clean .
docker image list
```
### Minimizarea numărului de straturi
Am creat un `Dockerfile.few` cu următorul conținut:
```dockerfile
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y nginx && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Am construit imaginea cu numele mynginx:few și am verificat dimensiunea:
```bash
docker image build -t mynginx:few -f Dockerfile.few .
docker image list
```

### Utilizarea unei imagini de bază minime
Am creat un `Dockerfile.alpine` cu următorul conținut:
```dockerfile
# create from alpine image
FROM alpine:latest

# update system
RUN apk update && apk upgrade

# install nginx
RUN apk add nginx

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```
Am construit imaginea cu numele mynginx:alpine și am verificat dimensiunea:
```bash
docker image build -t mynginx:alpine -f Dockerfile.alpine .
docker image list
```
### Repachetarea imaginii
 Am repachetat imaginea  mynginx:raw in mynginx:repack:
 ```bash
docker container create --name mynginx mynginx:raw
docker container export mynginx | docker image import - mynginx:repack
docker container rm mynginx
docker image list
 ```
### Utilizarea tuturor metodelor
Am creat imaginea mynginx:minx utilizând toate metodele de optimizare. Pentru aceasta am definit următorul Dockerfile.min:

```dockerfile
# create from alpine image
FROM alpine:latest

# update system, install nginx and clean
RUN apk update && apk upgrade && \
    apk add nginx && \
    rm -rf /var/cache/apk/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```
Am construit imaginea cu numele mynginx:minx și am verificat dimensiunea. Am repachetat imaginea mynginx:minx în mynginx:min:
```bash
docker image build -t mynginx:minx -f Dockerfile.min .
docker container create --name mynginx mynginx:minx
docker container export mynginx | docker image import - myngin:min
docker container rm mynginx
docker image list
```

## Pornire și testare
Am verificat dimensiunile imaginilor: 
```bash
docker image list
```
```bash

REPOSITORY            TAG                  SIZE
myngin                min                  14.3MB
mynginx               minx                 14.4MB
mynginx               repack               211MB
mynginx               alpine               19.5MB
mynginx               few                  186MB
mynginx               clean                269MB
mynginx               raw                  269MB
```
## Întrebări:
### 1.  Care metodă de optimizare a imaginilor vi se pare cea mai eficientă?
Cea mai eficientă metodă este utilizarea unei imagini de bază minime (ex: Alpine), combinată cu curățarea cache-ului și reducerea numărului de straturi.

### 2.  De ce curățirea cache-ului pachetelor într-un strat separat nu reduce dimensiunea imaginii?
În Docker, fiecare RUN creează un strat nou. Dacă curățăm cache-ul într-un strat separat, dimensiunea cache-ului este deja salvată în stratul anterior și nu se elimină fizic din imagine. De aceea, trebuie să combinăm instalarea și curățarea în același strat.

### 3. Ce este repachetarea imaginii?
Repachetarea imaginii înseamnă exportarea unui container într-un fișier tar și importarea lui ca o imagine nouă, astfel eliminând metadatele Docker inutile și micșorând imaginea.

## Concluzii:
* Utilizarea unei imagini de bază mai mici (ex: Alpine) are cel mai mare impact asupra dimensiunii finale.
* Combinarea mai multor optimizări (curățare, reducerea straturilor, imagini minime) produce imagini extrem de compacte și eficiente.