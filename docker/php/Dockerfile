FROM php:8.2-fpm

# 1. 阿里云 APT 源 (推荐直接写入内容，避免本地文件版本不一致)
# 注意：PHP 8.2-fpm 默认基于 Debian 11 (bullseye)，请确保源匹配
RUN echo "deb http://mirrors.aliyun.com/debian/ bullseye main non-free contrib" > /etc/apt/sources.list \
    && echo "deb http://mirrors.aliyun.com/debian-security/ bullseye-security main" >> /etc/apt/sources.list \
    && echo "deb http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib" >> /etc/apt/sources.list

# 2. 系统依赖
RUN apt-get update && apt-get install -y \
    git curl zip unzip \
    libpng-dev libjpeg62-turbo-dev libfreetype-dev \
    libzip-dev libicu-dev libonig-dev \
    && rm -rf /var/lib/apt/lists/*

# 3. GD & 基础扩展
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
        pdo_mysql mbstring exif pcntl bcmath gd intl zip opcache

# 4. Redis扩展 (推荐使用手动编译方式，避免 PECL 弃用风险)
# 方案：下载 Redis 源码手动编译，这是最兼容的方式
RUN curl -fsSL https://github.com/phpredis/phpredis/archive/develop.zip -o /tmp/redis.zip \
    && unzip /tmp/redis.zip -d /tmp \
    && rm /tmp/redis.zip \
    && cd /tmp/phpredis-develop \
    && phpize \
    && ./configure \
    && make -j$(nproc) \
    && make install \
    && docker-php-ext-enable redis

# 5. 开发环境opcache（改了revalidate_freq=2，改代码实时刷新）
RUN echo "opcache.enable=1" > /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.memory_consumption=256" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.validate_timestamps=1" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.revalidate_freq=2" >> /usr/local/etc/php/conf.d/opcache.ini

# 6. Composer 阿里镜像 (推荐使用官方推荐的最新版本)
RUN curl -o /usr/bin/composer.phar -L https://mirrors.aliyun.com/composer/composer.phar \
    && chmod +x /usr/bin/composer.phar \
    && mv /usr/bin/composer.phar /usr/bin/composer \
    # 更新到最新版本并配置阿里云镜像
    && composer self-update \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

# 7. 适配Mac权限用户 uid=1000
RUN groupadd -g 1000 www && useradd -u 1000 -g www -m www

# 提前创建工作目录，并将所有权赋予 www 用户
# 这样即使 Docker 挂载了命名卷，属主也会保持为 www
RUN mkdir -p /var/www/html && chown -R www:www /var/www/html

WORKDIR /var/www/html

# 保持这行！这是安全底线
USER www
