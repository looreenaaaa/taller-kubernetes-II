FROM node:18-alpine
LABEL authors="lorenappff"

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# puerto por defecto
EXPOSE 3000

CMD ["npm", "start"]

ENTRYPOINT ["top", "-b"]