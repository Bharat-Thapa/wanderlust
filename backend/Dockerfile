# Stage 1: Build the application
FROM node:21 AS builder
WORKDIR /app
COPY . .
RUN npm install

# Stage 2: Create the production image
FROM node:21-slim AS production
WORKDIR /app
COPY --from=builder /app .
RUN npm install
RUN npm run test
COPY .env.docker .env
EXPOSE 5000
CMD ["npm", "start"]
