# Iniciando um novo projeto utilizando o docker-compose

1. Crie uma pasta para sua aplicação com os arquivos iniciais:

```
$ mkdir myapp && cd myapp
$ touch Dockerfile Gemfile Gemfile.lock entrypoint.sh docker-compose.yml
```

2. No arquivo **Dockerfile**, insira este conteúdo:

```
FROM ruby:2.6.3
RUN curl https://deb.nodesource.com/setup_14.x | bash
RUN curl https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install -y gcc g++ make nodejs yarn
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev
RUN mkdir /myapp

WORKDIR /myapp
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
COPY . /myapp

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]
```

3. No arquivo **Gemfile**, insira este conteúdo:

```
source 'https://rubygems.org'
gem 'rails', '6.0.2.2'
```

4. No arquivo **entrypoint.sh**, insira este conteúdo:

```
#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /myapp/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```

5. No arquivo **docker-compose.yml**, insira este conteúdo:

```
version: '3.6'

volumes:
  gems-app:

services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: "db"
      POSTGRES_HOST_AUTH_METHOD: "trust"

  webpacker:
    build: .
    command: bash -c "rm -rf /myapp/public/packs; /myapp/bin/webpack-dev-server"
    volumes:
      - .:/myapp
    ports:
      - "3035:3035"

  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/myapp
      - gems-app:/usr/local/bundle
    ports:
      - "3000:3000"
    depends_on:
      - db
```

6. Criar o projeto

```
$ sudo docker-compose run --rm web rails new . -T --force --database=postgresql
```

7. Configurando o banco de dados `database.yml`

```
development: &default
  adapter: postgresql
  database: myapp_development
  encoding: unicode
  host: db
  username: postgres
  password:
  pool: <%= ENV.fetch('RAILS_MAX_THREADS') { 5 } %>

test:
  <<: *default
  database: myapp_test

production:
  <<: *default
  database: myapp_production
```

8. Criando o banco de dados

```
$ docker-compose run --rm web rails db:create
```

9. Subindo seu servidor

```
$ docker-compose up web
```

10. Acessando banco de dados

```
$ docker-compose run --rm db psql -h db -U postgres

Ou:

$ docker-compose run --rm db psql -d postgres://postgres@db/myapp_development
```

11. Trocando propriedade dos arquivos:

```
$ sudo chown -R $USER:$USER .
```

12. Acessar o endereço `localhost:3000`

13. Instalando React (opcional)

```
$ docker-compose run --rm web rails webpacker:install:react

<%= javascript_pack_tag 'hello_react' %>
```

14. Editar a configuração host do webpacker `config/webpacker.yml`

```
dev_server:
  host: 0.0.0.0
```

15. Editar o arquivo `config/environments/development.rb`

```
# Add to whitelist the '172.18.0.1' network space in the Web Console config.
config.web_console.whitelisted_ips = ['192.168.0.0/16', '172.0.0.0/8']
```

[Referência](https://medium.com/@pedro_ayres/rails-6-docker-postgres-a0158b01bfde)
