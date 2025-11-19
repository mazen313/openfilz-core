# Multi-stage Dockerfile for OpenFilz Web UI
# Stage 1: Build the Angular application
FROM node:22-alpine AS build
ARG NG_APP_API_URL

ARG NG_APP_GRAPHQL_URL
# Set the environment variables
ENV NG_APP_API_URL=https://openfilzapi.openfilz.org/api/v1
ARG NG_APP_GRAPHQL_URL=https://openfilzapi.openfilz.org/graphql/v1

WORKDIR /app

# Copy package files
COPY openfilz-web/package.json openfilz-web/package-lock.json ./

# Install dependencies
RUN npm ci --legacy-peer-deps

# Copy source code
COPY openfilz-web/ .

RUN echo 'console.log("NG_APP_GRAPHQL_URL", import.meta.env["NG_APP_GRAPHQL_URL"])' >>src/main.ts
# Build the application for production
RUN npm run build -- --configuration production

# Stage 2: Serve with Nginx
FROM nginxinc/nginx-unprivileged:alpine-perl

# Copy built assets from build stage
COPY --from=build /app/dist/openfilz-ui/browser /usr/share/nginx/html

# Copy nginx configuration
COPY openfilz-web/nginx/default.conf /etc/nginx/conf.d/default.conf

# Switch to nginx user
USER nginx

# Expose port
EXPOSE 8080

# Health check
# HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
#   CMD wget --no-verbose --tries=1 --spider http://localhost:8080/ || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
