FROM node:14

RUN apt update && apt upgrade -y
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD node index.js
EXPOSE 4000