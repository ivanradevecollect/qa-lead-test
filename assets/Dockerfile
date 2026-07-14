FROM node:24 AS base

# Install dependencies only when needed
FROM base AS deps
WORKDIR /app

# Copy package files
COPY package.json package-lock.json* ./
RUN npm ci

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects anonymous telemetry data about general usage.
ENV NEXT_TELEMETRY_DISABLED=1

# With `output: 'standalone'`, next.config.js is evaluated at BUILD time and
# its rewrites() destinations are baked into the build output. These URLs must
# therefore be present during `npm run build`, not just at runtime. Defaults
# match the docker-compose service names; override via build args if needed.
ARG BOOKING_API=http://rbp-booking:3000
ARG ROOM_API=http://rbp-room:3001
ARG BRANDING_API=http://rbp-branding:3002
ARG AUTH_API=http://rbp-auth:3004
ARG REPORT_API=http://rbp-report:3005
ARG MESSAGE_API=http://rbp-message:3006
ENV BOOKING_API=$BOOKING_API \
    ROOM_API=$ROOM_API \
    BRANDING_API=$BRANDING_API \
    AUTH_API=$AUTH_API \
    REPORT_API=$REPORT_API \
    MESSAGE_API=$MESSAGE_API

RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

# Runtime env for the app's API route handlers (read per-request). These can
# still be overridden at runtime, but note the rewrite destinations above are
# fixed at build time — see comment in the builder stage.
ENV BOOKING_API=http://rbp-booking:3000
ENV ROOM_API=http://rbp-room:3001
ENV BRANDING_API=http://rbp-branding:3002
ENV AUTH_API=http://rbp-auth:3004
ENV MESSAGE_API=http://rbp-message:3006
ENV REPORT_API=http://rbp-report:3005

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

RUN mkdir -p /app/.next/cache && chown -R nextjs:nodejs /app/.next
USER nextjs

EXPOSE 80

ENV PORT=80
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]
