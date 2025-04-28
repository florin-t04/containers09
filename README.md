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

## Pașii de realizare

### 1. Pregătirea mediului
- A fost instalat Docker pe calculator.
- A fost creat un repository `containers09`.
- În directorul `containers09` a fost creat un subdirector `site` cu fișierele html, css și js ale site-ului.

### 2. Crearea imaginii inițiale (Dockerfile.raw)
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





REPOSITORY            TAG           IMAGE ID       CREATED              SIZE
mynginx               clean         ac4a1f2ef422   10 seconds ago       269MB
mynginx               raw           402f639d57eb   About a minute ago   269MB

REPOSITORY            TAG           IMAGE ID       CREATED          SIZE
mynginx               few           f161ff4dc627   12 seconds ago   186MB
mynginx               clean         ac4a1f2ef422   2 minutes ago    269MB
mynginx               raw           402f639d57eb   3 minutes ago    269MB

REPOSITORY            TAG           IMAGE ID       CREATED              SIZE
mynginx               alpine        bd6c6ce736d4   9 seconds ago        19.5MB
mynginx               few           f161ff4dc627   About a minute ago   186MB
mynginx               clean         ac4a1f2ef422   3 minutes ago        269MB
mynginx               raw           402f639d57eb   4 minutes ago        269MB

REPOSITORY            TAG           IMAGE ID       CREATED          SIZE
mynginx               repack        8d7612f49fb0   20 seconds ago   211MB
mynginx               alpine        bd6c6ce736d4   5 minutes ago    19.5MB
mynginx               few           f161ff4dc627   6 minutes ago    186MB
mynginx               clean         ac4a1f2ef422   9 minutes ago    269MB
mynginx               raw           402f639d57eb   10 minutes ago   269MB

REPOSITORY            TAG           IMAGE ID       CREATED              SIZE
myngin                min           ab443a785689   21 seconds ago       14.3MB
mynginx               minx          992b4d42ba3d   About a minute ago   14.4MB
mynginx               repack        8d7612f49fb0   3 minutes ago        211MB
mynginx               alpine        bd6c6ce736d4   8 minutes ago        19.5MB
mynginx               few           f161ff4dc627   9 minutes ago        186MB
mynginx               clean         ac4a1f2ef422   11 minutes ago       269MB
mynginx               raw           402f639d57eb   13 minutes ago       269MB