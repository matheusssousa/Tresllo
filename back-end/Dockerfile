# --- Estágio 1: Dependências (PHP + Composer) ---
FROM composer:2 AS vendor

WORKDIR /app
COPY composer.json composer.lock ./
# Instala SÓ dependências de produção
RUN composer install --no-dev --no-interaction --no-scripts --no-autoloader --prefer-dist

# --- Estágio 2: Imagem Base (PHP + Apache) ---
FROM php:8.2-apache

ENV APACHE_DOCUMENT_ROOT /var/www/html/public

# Instala extensões PHP essenciais para o Laravel
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libzip-dev \
    unzip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd pdo pdo_mysql zip bcmath \
    && a2enmod rewrite

# Copia os arquivos de configuração do Apache
COPY 000-default.conf /etc/apache2/sites-available/000-default.conf

# --- Estágio 3: Aplicação ---
WORKDIR /var/www/html

# Copia as dependências do Estágio 1
COPY --from=vendor /app/vendor/ ./vendor/

# Copia o resto do código do Laravel
COPY . .

# Gera o autoloader otimizado do Composer
RUN composer dump-autoload --no-dev --optimize

# Ajusta permissões
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache
RUN chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache