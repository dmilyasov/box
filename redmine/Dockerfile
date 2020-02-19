FROM ubuntu:latest

#установка ruby согласно сайту https://gorails.com/setup/ubuntu/18.04

RUN apt install -y curl 
RUN curl -sL https://deb.nodesource.com/setup_12.x | bash -
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg |apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update
RUN apt-get install -y git-core zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev nodejs yarn
RUN cd
RUN git clone https://github.com/rbenv/rbenv.git ~/.rbenv
RUN echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
RUN echo 'eval "$(rbenv init -)"' >> ~/.bashrc
RUN exec $SHELL
RUN git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
RUN echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
RUN exec $SHELL
RUN rbenv install 2.3.0
RUN rbenv global 2.3.0

# установка redmine
RUN mkdir -p /var/www/
VOLUME ["/var/www/"]
RUN cd /var/www/
RUN svn co https://svn.redmine.org/redmine/branches/4.1-stable/ redmine

RUN cd redmine
RUN cp config/configuration.yml.example config/configuration.yml

RUN gem install bundler
RUN bundle install --without development test postgresql sqlite

#загрузка данных по умолчанию (согласно документации redmine: https://www.redmine.org/projects/redmine/wiki/RusRedmineInstall)

RUN bundle exec rake generate_secret_token
RUN RAILS_ENV=production bundle exec rake db:migrate
RUN RAILS_ENV=production REDMINE_LANG=ru rake redmine:load_default_data

# копируем файл настроек для подключения к mysql
COPY database.yml /var/www/redmine/config/database.yml

# установка passenger и nginx
RUN gem install passenger 
RUN passenger-install-nginx-module --auto

RUN echo 'include vhost/*.conf;' >> /opt/nginx/conf/nginx.conf
RUN mkdir -p /opt/nginx/conf/vhost

# копируем файлы настроек для подключения redmine+nginx

COPY redmine.conf /opt/nginx/conf/vhost/redmine.conf
COPY nginx.service /lib/systemd/system/nginx.service
