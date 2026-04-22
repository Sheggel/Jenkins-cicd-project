# ========================
# Stage 1: Build the React app
# ========================
FROM node:20-alpine AS builder
WORKDIR /app

# Copy package files first for better Docker layer caching
COPY package*.json ./

# Install dependencies (use ci for clean production install)
RUN npm ci --only=production=false

# Copy the rest of the application code
COPY . .

# Build the app (outputs to /app/dist for Vite)
RUN npm run build

# ========================
# Stage 2: Serve with Nginx (production runtime)
# ========================
FROM nginx:1.27-alpine AS production

# Remove default nginx config
RUN rm -rf /etc/nginx/conf.d/default.conf

# Copy custom nginx config for React SPA routing (important for React Router)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy the built static files from builder stage
COPY --from=builder /app/build /usr/share/nginx/html

# Optional: Add non-root user for better security (recommended in EKS)
RUN adduser -D -u 1001 nginxuser && \
    chown -R nginxuser:nginxuser /usr/share/nginx/html && \
    chown -R nginxuser:nginxuser /var/cache/nginx

USER nginxuser

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]