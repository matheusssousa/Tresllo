# --- Estágio 1: Dependências (PHP + Composer) ---
FROM composer:2 AS vendor

WORKDIR /app
COPY composer.json composer.lock ./
# (Aquele --no-autoloader estava errado, removi ele também)
RUN composer install --no-dev --no-interaction --no-scripts --prefer-dist

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

# <<< LINHA NOVA ABAIXO! <<<
# Copia o executável do Composer do primeiro estágio
COPY --from=vendor /usr/bin/composer /usr/bin/composer

# Copia as dependências do Estágio 1
COPY --from=vendor /app/vendor/ ./vendor/

# Copia o resto do código do Laravel
COPY . .

# Gera o autoloader otimizado do Composer (AGORA VAI FUNCIONAR)
RUN composer dump-autoload --no-dev --optimize

# Ajusta permissões
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache
RUN chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache