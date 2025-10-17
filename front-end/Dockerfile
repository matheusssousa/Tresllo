# --- Estágio 1: Dependências ---
FROM node:18-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install

# --- Estágio 2: Build ---
FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
# Seta a URL da API para o build de produção (sem http://localhost)
ENV NEXT_PUBLIC_API_URL=/api
RUN npm run build

# --- Estágio 3: Produção ---
FROM node:18-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production

# Copia o build do Estágio 2
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
CMD ["node", "server.js"]