FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
WORKDIR /code

COPY package.json pnpm-lock.yaml* ./
RUN yarn global add pnpm && pnpm i --frozen-lockfile
RUN pnpm add ts-node typescript

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /code
COPY --from=deps /code/node_modules ./node_modules

COPY . .

# Declare all build args are env vars for build
#ARG GITHUB_ID
#ENV GITHUB_ID ${GITHUB_ID}
#ARG GITHUB_SECRET
#ENV GITHUB_SECRET ${GITHUB_SECRET}
#ARG NEXTAUTH_SECRET
#ENV NEXTAUTH_SECRET ${NEXTAUTH_SECRET}
#ARG NEXTAUTH_URL
#ENV NEXTAUTH_URL ${NEXTAUTH_URL}
#ARG NEXT_PUBLIC_API_URL
#ENV NEXT_PUBLIC_API_URL ${NEXT_PUBLIC_API_URL}

RUN yarn run prisma:generate
RUN yarn run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /code

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /code/public ./public

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /code/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /code/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
# CMD ["node", "--inspect=0.0.0.0:9229", "-r", "ts-node/register", "server.js"]
