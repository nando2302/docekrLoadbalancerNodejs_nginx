# **Loadbalancer Nodejs menggunakan Nginx dan mengimplementasi dengan docker build** #

Load balancer adalah salah satu cara pembagian traffic padad suatu server dengan mendistribusikan traffic tersebut ke beberapa server dengan bantuan proxy seperti nginx dan Haproxy
    
Nginx adalah web server dengan performa yang andal dan mempunyai beberapa fitur canggih lain yang mudah dikonfigurasi. Alhasil, Nginx mampu membuat website Anda lebih powerful dan canggih. Pada awal munculnya, Nginx hanya dipakai untuk server HTTP saja. Seiring perkembangan teknologinya, sekarang Nginx juga dipakai sebagai HTTP cache, load balancer (HTTP, TCP, dan UDP), dan server proxy (IMAP, POP3, dan SMTP).

Node.js adalah platform  buatan Ryan Dahl untuk menjalankan aplikasi web berbasis JavaScript yang dikenalkan pada tahun 2009. Dengan platform ini, Anda dapat menjalankan JavaScript dari sisi server.

# **Arsitektur Loadbalancer Dengan menggunakan 2 web server** #
menggunakana 1 loadbalancer dan 2 web server dengan perbedaan port pada web server
![Arsitektur_loadbalancer]

# Installation #

buat folder dengan nama loadbalancer dan masuk kedalam loadbalancer tersebut seperti yang dibawah ini:
```sh
mkdir loadbalancer 
cd loadbalancer 
```

buat lah folder untuk menginstall nodejs dan express sebagain javascript dan server
```
mkdir nodejs 
cd nodejs
npm init -y 
npm install express --save || npm i express
npm install
touch index.js
touch Dockerfile
```
isi dari index.js seperti dibawah ini:
```
const express = require("express");
const app = express();
const PORT = process.env.PORT || 4000;
const server = process.env.server||"GateWay"
app.get("/", (req, res, next) => {
    res.send(`<h1>This Request redirect to the server ${server}</h1>`);
});
app.listen(PORT, () => {
    console.log(`Server Running at port:${PORT}`)
});
```
isi dari Dockerfile difolder nodejs 
```
FROM node:14
RUN apt update && apt upgrade -y
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD node index.js
EXPOSE 4000
```

selanjutnya build docker dengan command sebagai berikut:
```
docker build -t nodeserver-app:1.0 .
```
setelah build berhasil coba cek di docker images

selajutnya keluar dari folder nodejs dengan command dan buat folder nginx seperti berikut ini:
```
cd ..
mkdir nginx
 cd nginx
 touch nginx.conf
 touch Dockerfile
```

isi dari file nginx.conf sebagai berikut:
```
upstream loadbalance {
    server 172.17.0.1:4001;
    server 172.17.0.1:4002;
}

server {
    location / {
    proxy_pass http://loadbalance;
    }
}
```
isi dari file Dockerfile sebagai berikut:
```
FROM nginx

RUN apt update && apt upgrade -y

RUN rm /etc/nginx/conf.d/default.conf

COPY nginx.conf /etc/nginx/conf.d/default.conf
```
selanjut coba jalan build dockerfile dengan command berikut:
```
docker build -t nginx-loadbalancer:1.0 .
```
coba liat di docker images apakah sudah berhasil dibuild

selanjutnya buat 2 container untuk nodejs seperti berikut:
```
 docker container run -d -p 4001:4000 --name server-01 -e "server=satu" nodeserver-app:1.0.0

docker container run -d -p 4002:4000 --name server-02 -e "server=dua" nodeserver-app:1.0.0
```
selanjutnya buat 1 container untuk nginx seperti berikut:
```
docker container run -p 4000:80 -d nginx-loadbalancer:1.0
```



nah kalo kalian malas untuk menbuat ulang kalian clone aja github ini lalu jalan kan 3 beberapa command berikut 
```
docker build -t nodeserver-app:1.0 .
docker build -t nginx-loadbalancer:1.0 .

 docker container run -d -p 4001:4000 --name server-01 -e "server=satu" nodeserver-app:1.0.0

docker container run -d -p 4002:4000 --name server-02 -e "server=dua" nodeserver-app:1.0.0

docker container run -p 4000:80 -d nginx-loadbalancer:1.0
```

# Hasil dari #
![webserver-01]
![webserver-02]
