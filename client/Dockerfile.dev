FROM node:16-alpine as builder
WORKDIR /app
COPY ./package.json .
RUN npm install
RUN npm install -g @angular/cli
COPY . .
RUN npm run build -- --output-path=dist

FROM nginx:stable-alpine

RUN rm -rf /usr/share/nginx/html/*
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80

CMD [ "nginx", "-g", "daemon off;" ]