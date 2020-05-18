FROM ruby:2.6.1
RUN apt-get update -qq && apt-get install -y vim nodejs mysql-client
WORKDIR /tmp
ADD Gemfile Gemfile
RUN bundle install
COPY . /myapp
ENV APP_HOME /myapp
RUN mkdir -p ${APP_HOME}
WORKDIR ${APP_HOME}
ADD . ${APP_HOME}
