# ---- Builder Stage ----
# Use a full Node.js image for building
FROM node:20 AS builder

# Set working directory
WORKDIR /app

# Copy package files for shared dependencies
COPY ./shared/package*.json ./shared/
# Install shared dependencies
WORKDIR /app/shared
RUN npm ci

# Copy package files for agent dependencies
WORKDIR /app
COPY ./bytebot-agent/package*.json ./bytebot-agent/
# Install agent dependencies (including devDependencies)
WORKDIR /app/bytebot-agent
RUN npm ci

# Copy the rest of the source code
WORKDIR /app
COPY ./shared ./shared
COPY ./bytebot-agent/ ./bytebot-agent/

# Generate Prisma client
WORKDIR /app/bytebot-agent
RUN npx prisma generate

# Build the TypeScript project
RUN npm run build

# ---- Runtime Stage ----
# Use a slim, production-ready image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Create a non-root user for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy over production dependencies from the builder stage
# First, copy package.json files to install only production dependencies
COPY --from=builder /app/shared/package*.json ./shared/
COPY --from=builder /app/bytebot-agent/package*.json ./bytebot-agent/

# Install shared production dependencies
WORKDIR /app/shared
RUN npm ci --omit=dev

# Install agent production dependencies
WORKDIR /app/bytebot-agent
RUN npm ci --omit=dev

# Copy the compiled application code from the builder stage
COPY --from=builder /app/bytebot-agent/dist ./dist
# Copy prisma schema and generated client
COPY --from=builder /app/bytebot-agent/prisma ./prisma
COPY --from=builder /app/bytebot-agent/node_modules/.prisma/client ./node_modules/.prisma/client

# Change ownership of the files to the non-root user
USER appuser

# Expose the port the app runs on
EXPOSE 9991

# The command to run the application
CMD ["npm", "run", "start:prod"]
