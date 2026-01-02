FROM php:8.1.34-fpm AS build

# curlのみ試しで特定バージョン指定、他は最新をインストール
# curlのバージョンはapt show curlで確認可能。インストール時にバージョン指定する際はapt show [パッケージ名]で確認
RUN apt-get update -y && apt-get install -y --no-install-recommends \
    curl=8.14.1-2+deb13u2 \
    git \
    libonig-dev \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# composerインストールに必要なため、{}だけ記載したファイルをコピー
COPY ./composer.json /var/www/html

# ref https://qiita.com/vitway/items/660f9121a9c6afbf1047
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php -r "if (hash_file('sha384', 'composer-setup.php') === 'c8b085408188070d5f52bcfe4ecfbee5f727afa458b2573b8eaaf77b3419b0bf2768dc67c86944da1544f06fa544fd47') { echo 'Installer verified'.PHP_EOL; } else { echo 'Installer corrupt'.PHP_EOL; unlink('composer-setup.php'); exit(1); }" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/local/bin/composer \
    && composer install

FROM php:8.1.34-fpm AS deploy

LABEL version="1.0"

COPY --from=build /usr/local/bin/composer /usr/local/bin/composer

WORKDIR /var/www/html

EXPOSE 80/tcp

# 非rootユーザーでメインプロセス実行
RUN useradd www-user \
    && chown www-user:www-user /var/www/html
USER www-user