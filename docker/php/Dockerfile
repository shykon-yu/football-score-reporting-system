FROM php:8.2-fpm

# 1. 替换为阿里云 APT 源
# 请确保你已经在 docker/php/ 目录下创建了名为 sources.list 的文件
COPY sources.list /etc/apt/sources.list

# 2. 安装系统依赖 & 清理缓存 (合并命令，减小镜像体积)
RUN apt-get update && apt-get install -y \
    git curl zip unzip \
    libpng-dev libjpeg62-turbo-dev libfreetype-dev \
    libzip-dev libicu-dev libonig-dev \
    && rm -rf /var/lib/apt/lists/*

# 3. 安装 PHP 扩展
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
        pdo_mysql mbstring exif pcntl bcmath gd intl zip opcache

# 4. 安装 Redis 扩展
RUN pecl install redis && docker-php-ext-enable redis

# 5. 配置 OPcache
RUN echo "opcache.enable=1" > /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.memory_consumption=256" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.validate_timestamps=1" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.revalidate_freq=0" >> /usr/local/etc/php/conf.d/opcache.ini

# 6. 安装 Composer (使用阿里云离线包，最快最稳)
# 下载地址: https://mirrors.aliyun.com/composer/composer.phar
RUN curl -o /usr/bin/composer.phar -L https://mirrors.aliyun.com/composer/composer.phar \
    && chmod +x /usr/bin/composer.phar \
    && mv /usr/bin/composer.phar /usr/bin/composer

# 7. 设置阿里云 Composer 镜像
RUN composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

WORKDIR /var/www/html
