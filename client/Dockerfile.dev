FROM node:16-alpine
WORKDIR "/app"
COPY ./package.json ./
RUN npm install
RUN npm install -g @angular/cli
COPY . .
CMD ["npm", "run", "start"]