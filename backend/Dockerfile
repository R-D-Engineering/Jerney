# ---- Build stage ----
FROM node:20-alpine AS build
WORKDIR /app
COPY package.json package-lock.json* ./
# Update npm to latest to pick up patched node-tar (CVE-2026-29786, CVE-2026-31802)
RUN npm install -g npm@latest && \
    npm ci --only=production && \
    npm cache clean --force

# ---- Production stage ----
FROM node:20-alpine AS production

# Security: run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Security: remove unnecessary packages
# DL3018 fix: pin dumb-init to a specific version for reproducible builds
RUN apk --no-cache add dumb-init=1.2.5-r3

WORKDIR /app

# Copy only production dependencies from build stage
COPY --from=build /app/node_modules ./node_modules
COPY src/ ./src/

# Security: own files by non-root user
RUN chown -R appuser:appgroup /app

USER appuser

EXPOSE 5000

# Security: use dumb-init for proper signal handling (PID 1 problem)
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/index.js"]