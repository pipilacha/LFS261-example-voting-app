FROM node:19.0.1-alpine

WORKDIR /app 

COPY . . 

RUN npm install  && \ 
    npm audit fix 
    
EXPOSE 4000

CMD npm start 
