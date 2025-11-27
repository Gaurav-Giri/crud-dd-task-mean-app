# ---- Stage 1: Build the Node backend ----
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

EXPOSE 3000

# Start the backend server
CMD ["npm", "start"]
