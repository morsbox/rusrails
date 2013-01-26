# Продвинутая командная строка Rails

Более продвинутое использование командной строки сфокусировано на полезных (даже иногда удивляющих) опциях утилит, и подгонке утилит к вашим потребностям и особенностям рабочего процесса. Сейчас мы перечислим трюки из рукава Rails.

### Rails с базами данными и SCM

При создании нового приложения на Rails, можно выбрать, какой тип базы данных и какой тип системы управления исходным кодом (SCM) собирается использовать ваше приложение. Это сэкономит вам несколько минут и, конечно, несколько строк.

Давайте посмотрим, что могут сделать для нас опции `--git` и `--database=postgresql`:

```bash
$ mkdir gitapp
$ cd gitapp
$ git init
Initialized empty Git repository in .git/
$ rails new . --git --database=postgresql
      exists
      create  app/controllers
      create  app/helpers
...
...
      create  tmp/cache
      create  tmp/pids
      create  Rakefile
add 'Rakefile'
      create  README.rdoc
add 'README.rdoc'
      create  app/controllers/application_controller.rb
add 'app/controllers/application_controller.rb'
      create  app/helpers/application_helper.rb
...
      create  log/test.log
add 'log/test.log'
```

Мы создали директорию **gitapp** и инициализировали пустой репозиторий перед тем, как Rails добавил бы созданные им файлы в наш репозиторий. Давайте взглянем, что он нам поместил в конфигурацию базы данных:

```bash
$ cat config/database.yml
# PostgreSQL. Versions 8.2 and up are supported.
#
# Install the ruby-postgres driver:
#   gem install ruby-postgres
# On Mac OS X:
#   gem install ruby-postgres -- --include=/usr/local/pgsql
# On Windows:
#   gem install ruby-postgres
#       Choose the win32 build.
#       Install PostgreSQL and put its /bin directory on your path.
development:
  adapter: postgresql
  encoding: unicode
  database: gitapp_development
  pool: 5
  username: gitapp
  password:
...
...
```

Она также создала несколько строчек в нашей конфигурации database.yml, соответствующих нашему выбору PostgreSQL как базы данных. Единственная хитрость с использованием опции SCM состоит в том, что сначала нужно создать директорию для приложения, затем инициализировать ваш SCM, и лишь затем можно запустить команду `rails new` для создания основы вашего приложения.

### `server` с различным бэкендом

Создано много различных веб серверов на Ruby, и многие из них могут использоваться для запуска Rails. С версии 2.3, Rails использует Rack для обслуживания своих веб-страниц, который означает, что может быть использован любой вебсервер, реализующий обработчик Rack. Они включают в себя WEBrick, Mongrel, Thin и Phusion Passenger (и это только немногие!).

NOTE: Детальнее о интеграции Rack, смотрите [Rails on Rack](/different-guides/rails-on-rack).

Чтобы использовать другой сервер, всего лишь установите его гем, затем используйте его имя, как первый параметр для `rails server`:

```bash
$ sudo gem install mongrel
Building native extensions.  This could take a while...
Building native extensions.  This could take a while...
Successfully installed gem_plugin-0.2.3
Successfully installed fastthread-1.0.1
Successfully installed cgi_multipart_eof_fix-2.5.0
Successfully installed mongrel-1.1.5
...
...
Installing RDoc documentation for mongrel-1.1.5...
$ rails server mongrel
=> Booting Mongrel (use 'rails server webrick' to force WEBrick)
=> Rails 3.1.0 application starting on http://0.0.0.0:3000
...
```
