FROM node:21

WORKDIR /usr/local/app

COPY package*.json ./

RUN npm install -g nodemon

# clean install
RUN npm ci

RUN npm cache clean --force

# Move the node_modules directory to the root of the container
RUN mv /usr/local/app/node_modules /node_modules

COPY . .

ENV PORT 3000

EXPOSE 3000

CMD ["node", "server.js"]
