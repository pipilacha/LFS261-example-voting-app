#FROM node:19.0.1-alpine
FROM node:8.9.0-alpine

WORKDIR /app 

COPY . . 

RUN npm install
#RUN npm install  && \
#npm audit fix --force
    
EXPOSE 4000

CMD npm start 
