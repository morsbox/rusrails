Руководство по обновлению Ruby on Rails
=======================================

Это руководство раскрывает шаги, которые нужно сделать, чтобы обновить свое приложение на новую версию Ruby on Rails. Эти шаги также доступны в отдельных руководствах по релизам.

Общий совет
-----------

Перед попыткой обновить существующее приложение, сперва следует убедиться, что есть хорошая причина для обновления. Нужно соблюсти баланс между несколькими факторами: необходимостью в новых особенностях, увеличением сложности в поиске поддержки для старого кода, доступностью вашего времени и навыков - это только некоторые из многих.

### Тестовое покрытие

Лучшим способом убедиться, что ваше приложение продолжает работать после обновления, это иметь хорошее тестовое покрытие до начала обновления. Если у вас нет автоматических тестов, проверяющих большую часть вашего приложения, тогда нужно потратить время, проверяя все части, которые изменились. В случае обновления Rails это означает каждый отдельный кусок функционала приложения. Пожалейте себя и убедитесь в хорошем тестовом покрытии _до_ начала обновления.

### Процесс обновления

При изменении версий Rails лучше двигаться медленно, одна минорная версия за раз, чтобы результативно использовать предупреждения об устаревании. Версии Rails записываются в форме Major.Minor.Patch. В версиях Major и Minor допустимо делать изменения в публичном API, и это может вызвать ошибки в вашем приложении. Версии Patch включают только исправления ошибок и не изменяют публичное API.

Процесс должен быть следующим:

1. Пишете тесты и убеждаетесь, что они проходят.
2. Переходите на последнюю версию patch, следующую после вашей текущей версии.
3. Чините тесты и устаревшие особенности.
4. Переходите на последнюю версию patch следующей версии minor.

Повторяйте этот процесс, пока не достигнете целевой версии Rails. Каждый раз, как вы переходите на новую версию, необходимо изменить номер версии Rails в Gemfile (и, возможно, версии других гемов) и запустить `bundle update`. Затем запустите задачу Update, упомянутую ниже, чтобы обновить конфигурационные файлы, затем запустите свои тесты.

Полный список всех выпущенных версий Rails можно найти [тут](https://rubygems.org/gems/rails/versions).

### Версии Ruby

В основном Rails использует последние выпущенные версии Ruby:

* Rails 5 требует Ruby 2.2.2 или новее.
* Rails 4 предпочитает Ruby 2.0 и требует Ruby 1.9.3 или новее.
* Rails 3.2.x это последняя ветка с поддержкой Ruby 1.8.7.
* Rails 3 и выше требует Ruby 1.8.7 или выше. Поддержка всех прежних версий Ruby была официально прекращена. Следует обновиться как можно быстрее.

TIP: В Ruby 1.8.7 p248 и p249 имеются ошибки маршализации, ломающие Rails. Хотя в Ruby Enterprise Edition это было исправлено, начиная с релиза 1.8.7-2010.02. В ветке 1.9, Ruby 1.9.1 не пригоден к использованию, поскольку он иногда вылетает, поэтому, если хотите использовать 1.9.x перепрыгивайте сразу на 1.9.3 для гладкой работы.

### Задача Update

Rails предоставляет задачу `app:update` (`rake rails:update` на 4.2 и раньше). После обновления версии Rails в Gemfile, запустите эту задачу. Она поможет вам с созданием новых файлов и изменением старых файлов в интерактивной сессии.

```bash
$ rails app:update
   identical  config/boot.rb
       exist  config
    conflict  config/routes.rb
Overwrite /myapp/config/routes.rb? (enter "h" for help) [Ynaqdh]
       force  config/routes.rb
    conflict  config/application.rb
Overwrite /myapp/config/application.rb? (enter "h" for help) [Ynaqdh]
       force  config/application.rb
    conflict  config/environment.rb
...
```

Не забывайте просматривать разницу, чтобы увидеть какие-либо неожидаемые изменения.

(Upgrading from Rails 5.0 to Rails 5.1) Обновление с Rails 5.0 на Rails 5.1
-------------------------------------

Подробнее о внесенных изменениях в Rails 5.1 смотрите в [заметках о релизе](/5_1_release_notes).

### Высокоуровневый `HashWithIndifferentAccess` в скором времени устареет.

Если ваше приложение использует высокоуровневый класс `HashWithIndifferentAccess`, то вам следует вместо него постепенно переходить на `ActiveSupport::HashWithIndifferentAccess`.

Это всего лишь постепенное устаревание, которое означает, что ваш код не будет прерываться в данный момент и не будет отображаться предостережение об устаревании, но эта константа в будущем будет удалена.

Кроме того, если у вас есть довольно старые документы YAML, содержащие дампы таких объектов, вам может понадобиться загрузить и снова сбросить их, чтобы убедиться, что они ссылаются на нужную константу и что их загрузка не будет прерываться в будущем.

### `application.secrets` теперь загружается со всеми ключами в качестве символов

Если ваше приложение хранит вложенную конфигурацию в `config/secrets.yml`, все ключи теперь загружаются как символы, поэтому доступ с использованием строк должен быть изменен.

С:

```ruby
Rails.application.secrets[:smtp_settings]["address"]
```

На:

```ruby
Rails.application.secrets[:smtp_settings][:address]
```

(Upgrading from Rails 4.2 to Rails 5.0) Обновление с Rails 4.2 на Rails 5.0
---------------------------------------------------------------------------

Подробнее о внесенных изменениях в Rails 5.0 смотрите в [заметках о релизе](/5_0_release_notes).

### Требуется Ruby 2.2.2+

Начиная с Ruby on Rails 5.0, Ruby 2.2.2+ являются единственными поддерживаемыми версиями Ruby. Перед тем, как продолжить, убедитесь, что вы на Ruby версии 2.2.2 или выше.

### Active Record models now inherit from ApplicationRecord by default

В Rails 4.2 модель Active Record наследуется от `ActiveRecord::Base`. В Rails 5.0 все модели наследуются от `ApplicationRecord`.

`ApplicationRecord` - это новый суперкласс для всех моделей приложения, аналогично контроллерам, наследуемым от `ApplicationController` вместо `ActionController::Base`. Это дает приложению единое место для настройки специфичного для приложения поведения моделей.

При обновлении с Rails 4.2 до Rails 5.0 необходимо создать файл `application_record.rb` в `app/models/` и добавить следующее содержимое:

```
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true
end
```

Затем убедитесь, что все ваши модели наследуются от него.

### Прерывание цепочек колбэков с помощью `throw(:abort)`

В Rails 4.2 в Active Record и Active Model, когда колбэк 'before' возвращает `false`, вся цепочка цепочка колбэков прерывалась. Другими словами, последующие колбэки 'before' не выполнялись, как и действие, обернутое в колбэки.

В Rails 5.0, возврат `false` колбэком Active Record или Active Model не будет иметь этого побочного эффекта прерывания цепочки колбэков. Вместо этого, цепочки колбэков должны быть явно прерваны вызовом `throw(:abort)`.

При обновлении с Rails 4.2 на Rails 5.0, возврат `false` в этих типах колбэков все еще будет прерывать цепочку колбэков, но вы получите предостережение об устаревании об этом грядущем изменении.

Когда вы будете готовы, можно переключиться на новое поведение и убрать предостережение об устаревании, добавив следующую конфигурацию в `config/application.rb`:

    ActiveSupport.halt_callback_chains_on_return_false = false

Отметим, что эта опция не влияет на колбэки Active Support, так как они никогда не прерывались в зависимости от возвращаемого значения.

За подробностями обратитесь к [#17227](https://github.com/rails/rails/pull/17227).

### ActiveJob теперь по умолчанию наследуется от ApplicationJob

В Rails 4.2 Active Job наследуется от `ActiveJob::Base`. В Rails 5.0 это поведение было изменено на наследование от `ApplicationJob`.

При обновлении с Rails 4.2 на Rails 5.0 необходимо создать файл `application_job.rb` в `app/jobs/` со следующим содержимым:

```
class ApplicationJob < ActiveJob::Base
end
```

Затем убедитесь, что все классы заданий наследуются от него.

За подробностями обратитесь к [#19034](https://github.com/rails/rails/pull/19034).

### Тестирование контроллеров Rails

#### Извлечение некоторых методов хелпера в `rails-controller-testing`

`assigns` и `assert_template` были извлечены в гем `rails-controller-testing`. Чтобы продолжить использование этих методов, добавьте `gem 'rails-controller-testing'` в свой Gemfile.

Если вы используете Rspec для тестирования, обратитесь к документации гема по требуемой дополнительной конфигурации.

#### Новое поведение при загрузке файлов

Если вы используете `ActionDispatch::Http::UploadedFile` в своих тестах для загрузки файлов, то вам нужно будет использовать вместо него аналогичный класс `Rack::Test::UploadedFile`.

За подробностями обратитесь к [#26404](https://github.com/rails/rails/issues/26404).

### Автозагрузка отключена после загрузки в среде production

Автозагрузка теперь отключена после загрузки в среде production по умолчанию.

Нетерпеливая загрузка приложения - это часть процесса загрузки, поэтому константы верхнего уровня все еще автоматически загружаются, их файлы не нужно требовать.

Более глубокие константы запускаются только во время выполнения, как содержимое обычных методов, и это нормально, так как содержащий их файл был нетерпеливо загружен при начальной загрузке.

Для абсолютного большинства приложений это изменение не требует каких-либо действий. Но, в очень редком случае, когда вашему приложению требуется автозагрузка в режиме production, установите `Rails.application.config.enable_dependency_loading` в true.

### Сериализация в XML

`ActiveModel::Serializers::Xml` был извлечен из Rails в гем `activemodel-serializers-xml`. Чтобы продолжить сериализацию в XML в вашем приложении, добавьте `gem 'activemodel-serializers-xml'`в свой Gemfile.

### Убрана поддержка старой версии адаптера `mysql`

В Rails 5 убрана поддержка старой версии адаптера БД `mysql`. Большинство пользователей могут использовать вместо него `mysql2`. Он будет сконвертирован в отдельный гем, если найдется кто-то для его поддержки.

### Убрана поддержка Debugger

`debugger` не поддерживается Ruby 2.2, который требуется для Rails 5. Вместо него используйте `byebug`.

### Использование bin/rails для запуска задач и тестов

Rails 5 добавляет возможность запускать задачи и тесты с помощью `bin/rails` вместо rake. В основном эти изменения были внесены параллельно с rake, но некоторые были перенесены полностью.

Чтобы запустить тесты, просто напишите `bin/rails test`.

`rake dev:cache` теперь `rails dev:cache`.

Запустите `bin/rails`, чтобы просмотреть список доступных команд.

### `ActionController::Parameters` больше не наследуется от `HashWithIndifferentAccess`

Вызов `params` в вашем приложении теперь возвращает объект, а не хэш. Если ваши параметры уже дозволены, вам не нужно вносить каких-либо изменений. Если вы используете `slice` и другие методы, зависящие от возможности читать хэш, не смотря на `permitted?`, нужно обновить ваше приложение, чтобы сначала сделать permit, а затем конвертировать в хэш.

    params.permit([:proceed_to, :return_to]).to_h

### Теперь в `protect_from_forgery` по умолчанию `prepend: false`

`protect_from_forgery` по умолчанию `prepend: false`, что означает, что он буден вставлен в цепочку колбэков в том месте, в котором он вызывается в вашем приложении. Если вы хотите, чтобы `protect_from_forgery` всегда запускался первым, следует изменить приложение, чтобы использовался `protect_from_forgery prepend: true`.

### Обработчик шаблона по умолчанию теперь RAW

Файлы без обработчика шаблона в их расширении будут рендериться с помощью обработчика raw. Раньше Rails рендерил бы файлы с помощью обработчика шаблона ERB.

Если вы не хотите, чтобы ваш файл обрабатывался с помощью обработчика raw, следует добавить расширение к своему файлу, чтобы он анализировался подходящим обработчиком шаблона.

### Добавлено совпадение с подстановкой для зависимостей шаблонов

Теперь можно использовать совпадение с подстановкой для зависимостей ваших шаблонов. Например, если вы определяли ваши шаблоны так:

```erb
<% # Template Dependency: recordings/threads/events/subscribers_changed %>
<% # Template Dependency: recordings/threads/events/completed %>
<% # Template Dependency: recordings/threads/events/uncompleted %>
```

Теперь можно вызвать зависимость единожды с помощью символа подстановки.

```erb
<% # Template Dependency: recordings/threads/events/* %>
```

### Убрана поддержка гема `protected_attributes`

Гем `protected_attributes` больше не поддерживается в Rails 5.

### Убрана поддержка гема `activerecord-deprecated_finders`

Гем `activerecord-deprecated_finders` больше не поддерживается в Rails 5.

### Порядок тестов `ActiveSupport::TestCase` по умолчанию сейчас случайный

Когда в вашем приложении запускаются тесты, порядок по умолчанию сейчас `:random` вместо `:sorted`. Используйте следующую конфигурационную опцию, чтобы установить `:sorted` обратно.

```ruby
# config/environments/test.rb
Rails.application.configure do
  config.active_support.test_order = :sorted
end
```

### `ActionController::Live` стал `Concern`

Если вы включали `ActionController::Live` в другой модуль, который включался в ваш контроллер, то вы также должны расширить модуль с помощью `ActiveSupport::Concern`. Альтернативно можно использовать хук `self.included`, чтобы включить `ActionController::Live` непосредственно в контроллер, как только включится `StreamingSupport`.

Это означает, что если в вашем приложении должен быть собственный модуль потоковой передачи, следующий код сломается в режиме production:

```ruby
# Это обертка для потоковых контроллеров, выполняющих аутентификацию с помощью Warden/Devise.
# Смотрите https://github.com/plataformatec/devise/issues/2332
# Другим решением является аутентификация в роутере, как предложено в тщй проблеме
class StreamingSupport
  include ActionController::Live # это не будет работать в production для Rails 5
  # extend ActiveSupport::Concern # пока вы не раскомментируете эту строчку.

  def process(name)
    super(name)
  rescue ArgumentError => e
    if e.message == 'uncaught throw :warden'
      throw :warden
    else
      raise e
    end
  end
end
```

### Новые значения по умолчанию фреймворка

#### Опция Active Record для `belongs_to` - требуется по умолчанию

`belongs_to` по умолчанию теперь вызывает ошибку валидации, если связь не присутствует.

Это можно отключить для каждой связи с помощью `optional: true`.

Это значение по умолчанию можно автоматически сконфигурировать в новых приложениях. Если существующее приложение хочет добавить эту особенность, ее нужно включить в инициализаторе.

    config.active_record.belongs_to_required_by_default = true

#### Токены CSRF для формы

Rails 5 теперь поддерживает токены CSRF для формы, чтобы ослабить атаки с помощью внедрения кода с использованием форм, созданными JavaScript. Со включенной опцией формы вашего приложения будут иметь собственный токен CSRF, который определен для экшна и метода этой формы.

    config.action_controller.per_form_csrf_tokens = true

#### Защита от подделки с помощью проверки домена

Можно настроить свое приложение для проверки, что заголовок HTTP `Origin` должен быть сверен с доменом сайта, в качестве дополнительной защиты от CSRF. Установите следующей настройке true:

    config.action_controller.forgery_protection_origin_check = true

#### Разрешена настройка имени очереди Action Mailer

Имя очереди рассыльщика по умолчанию `mailers`. Эта конфигурационная опция позволяет глобально изменить имя очереди. Установите следующее в своих настройках:

    config.action_mailer.deliver_later_queue_name = :new_queue_name

#### Поддержка кэширования фрагментов во вьюхах Action Mailer

Установите `config.action_mailer.perform_caching` в своих настройках, чтобы определить, должны ли вьюхи Action Mailer поддерживать кэширование.

    config.action_mailer.perform_caching = true

#### Настройка вывода `db:structure:dump`

Если вы используете `schema_search_path` или другие расширения PostgreSQL, можно контролировать, как выгружается схема. Установите `:all`, чтобы генерировать всю выгрузку, или `:schema_search_path`, чтобы генерировать из пути поиска схемы.

    config.active_record.dump_schemas = :all

#### Настройка опций SSL, чтобы включить HSTS с поддоменами

Установите следующее в своих настройках, чтобы включить HSTS при использовании поддоменов:

    config.ssl_options = { hsts: { subdomains: true } }

#### Сохранение временной зоны получателя

При использовании Ruby 2.4 можно сохранять временную зону получателя, когда вызывается `to_time`.

    ActiveSupport.to_time_preserves_timezone = false

(Upgrading from Rails 4.1 to Rails 4.2) Обновление с Rails 4.1 на Rails 4.2
---------------------------------------------------------------------------

### Web Console

Сначала добавьте `gem 'web-console', '~> 2.0'` в группу `:development` своего Gemfile и запустите `bundle install` (он не включится при обновлении Rails). Как только он будет установлен, можно просто оставлять обращение к хелперу консоли (т.е., `<%= console %>`) в любой вьюхе, в которой вы ее хотите включить. Консоль также предоставляется на любой странице ошибок в среде development.

### Responders

Метод `respond_with` и метод класса `respond_to` были извлечены в гем `responders`. Для их использования просто добавьте `gem 'responders', '~> 2.0'` в свой Gemfile. Вызовы `respond_with` и `respond_to` (на уровне класса) больше не будут работать без подключения гема `responders` к зависимостям:

```ruby
# app/controllers/users_controller.rb

class UsersController < ApplicationController
  respond_to :html, :json

  def show
    @user = User.find(params[:id])
    respond_with @user
  end
end
```

Метод экземпляра `respond_to` не затронут и не требует дополнительного гема:

```ruby
# app/controllers/users_controller.rb

class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
    respond_to do |format|
      format.html
      format.json { render json: @user }
    end
  end
end
```

Подробнее смотрите [#16526](https://github.com/rails/rails/pull/16526).

### Обработка ошибок в транзакционных колбэках

В настоящий момент Active Record замалчивает ошибки, вызванные в колбэках `after_rollback` или `after_commit`, и только пишет их в логи. В следующей версии эти ошибки больше не будут замалчиваться. Вместо этого ошибки будут распространяться обычным образом, как в других колбэках Active
Record.

При определении колбэка `after_rollback` или `after_commit` вы получите предупреждение об устаревании об этом предстоящем изменении. Когда будете готовы, можно будет переключиться на новое поведение и убрать предупреждение об устаревании с помощью следующей настройки в вашем `config/application.rb`:

    config.active_record.raise_in_transactional_callbacks = true

Подробнее смотрите в [#14488](https://github.com/rails/rails/pull/14488) и [#16537](https://github.com/rails/rails/pull/16537).

### Упорядочивание тестовых случаев

В Rails 5.0 тестовые случаи будут по умолчанию запускаться в случайном порядке. В предвкушении этого изменения Rails 4.2 представил новую конфигурационную опцию `active_support.test_order` для явного указания упорядочивания тестов. Это позволит вам или заблокировать текущее поведение, установив этой опции `:sorted`, или переключиться на будущее поведение, установив этой опции `:random`.

Если не установить значение для этой опции, будет сформировано сообщение об устаревании. Чтобы его избежать, добавьте следующую строчку в своей тестовой среде:

```ruby
# config/environments/test.rb
Rails.application.configure do
  config.active_support.test_order = :sorted # или `:random`, если хотите
end
```

### Сериализованные атрибуты

При использовании произвольного кодировщика (например, `serialize :metadata, JSON`), назначение `nil` сериализованному атрибуту сохранит его в базу данных как `NULL`, вместо того, чтобы пропустить значение `nil` через кодировщик (например, `"null"` при использовании кодировщика `JSON`).

### Уровень лога в Production

В Rails 5 по умолчанию уровень лога для среды production будет изменен на `:debug` (с `:info`). Для сохранения текущего уровня по умолчанию, добавьте следующую строчку в свой `production.rb`:

```ruby
# Установите `:info`, соответствующий текущему значению по умолчанию, или
# ecnfyjdbnt `:debug` для переключения в будущее значение по умолчанию.
config.log_level = :info
```

### `after_bundle` в шаблонах Rails

Если у вас есть шаблон Rails, добавляющий все файлы в систему контроля версии, у него не получится добавить сгенерированные binstubs, так как это происходит до Bundler:

```ruby
# template.rb
generate(:scaffold, "person name:string")
route "root to: 'people#index'"
rake("db:migrate")

git :init
git add: "."
git commit: %Q{ -m 'Initial commit' }
```

Теперь можно обернуть вызовы `git` в блок `after_bundle`. Он запустится после того, как будут сгенерированы binstubs.

```ruby
# template.rb
generate(:scaffold, "person name:string")
route "root to: 'people#index'"
rake("db:migrate")

after_bundle do
  git :init
  git add: "."
  git commit: %Q{ -m 'Initial commit' }
end
```

### Санитайзер HTML в Rails

Появился выбор для санации фрагментов HTML в вашем приложении. Старый подход сканирования html официально устарел в пользу [`Rails HTML Sanitizer`](https://github.com/rails/rails-html-sanitizer).

Это означает, что методы `sanitize`, `sanitize_css`, `strip_tags` и `strip_links` теперь реализованы по-новому.

Новый санитайзер внутри использует [Loofah](https://github.com/flavorjones/loofah). Loofah, в свою очередь, использует Nokogiri, который является оберткой для парсеров, написанных на C и Java, таким образом, санирование должно стать быстрее, вне зависимости от того, на какой версии Ruby она запущена.

Новая версия обновляет `sanitize` таким образом, что он может принимать `Loofah::Scrubber` для мощной очистки. [Несколько примеров скраберов тут](https://github.com/flavorjones/loofah#loofahscrubber).

Также добавлены два новых скрабера: `PermitScrubber` и `TargetScrubber`. Подробнее читайте в [readme гема](https://github.com/rails/rails-html-sanitizer).

Документация для `PermitScrubber` и `TargetScrubber` объясняет, как можно получить полный контроль над тем, когда и как элементы должны быть отброшены.

Если вашему приложению необходима реализация старого санитайзера, включите `rails-deprecated_sanitizer` в свой Gemfile:

```ruby
gem 'rails-deprecated_sanitizer'
```

### Тестирование DOM в Rails

[Модуль `TagAssertions`](http://api.rubyonrails.org/classes/ActionDispatch/Assertions/TagAssertions.html) (содержащий методы, такие как `assert_tag`), [устарел](https://github.com/rails/rails/blob/6061472b8c310158a2a2e8e9a6b81a1aef6b60fe/actionpack/lib/action_dispatch/testing/assertions/dom.rb) в пользу методов `assert_select` из модуля `SelectorAssertions`, который был извлечен в [гем rails-dom-testing](https://github.com/rails/rails-dom-testing).

### Маскировка токенов аутентификации

В целях уменьшения атак SSL, `form_authenticity_token` теперь маскируется так, что он изменяется с каждым запросом. Таким образом, токены проверяются демаскируясь, а затем дешифруясь. Как последствие, любые стратегии проверки запросов из не-rails форм, которые полагались на статичный для сессии токен CSRF, должны принять это во внимание.

### Action Mailer

Раньше вызов метода рассыльщика на классе рассыльщика приводил к непосредственному запуску соответствующего метода экземпляра. С представлением Active Job и `#deliver_later`, это больше не так. В Rails 4.2 вызов методов экземпляра откладывается, пока не будут вызваны методы `deliver_now` или `deliver_later`. Например:

```ruby
class Notifier < ActionMailer::Base
  def notify(user, ...)
    puts "Called"
    mail(to: user.email, ...)
  end
end

mail = Notifier.notify(user, ...) # Notifier#notify в этом месте пока еще не вызван
mail = mail.deliver_now           # Напишет "Called"
```

Это не должно привести к каким-либо значимым изменениям для большинства приложений. Однако, если вам необходимо синхронно запустить некоторые не рассылающие методы, и раньше вы полагались на синхронное проксирующее поведение, вам следует объявить их как методы класса:

```ruby
class Notifier < ActionMailer::Base
  def self.broadcast_notifications(users, ...)
    users.each { |user| Notifier.notify(user, ...) }
  end
end
```

### Поддержка внешних ключей

DSL миграций был расширен поддержкой определений внешнего ключа. Если вы использовали гем Foreigner, рассмотрите его удаление. Отметьте, что поддержка внешних ключей в Rails это подмножество Foreigner. Это означает, что не каждое определение Foreigner может быть полностью заменено его аналогом DSL миграций Rails.

Процедура миграции следующая:

1. убрать `gem "foreigner"` из Gemfile.
2. запустить `bundle install`.
3. запустить `bin/rake db:schema:dump`.
4. убедиться, что `db/schema.rb` содержит каждое определение внешнего ключа с необходимыми опциями.


(upgrading-from-rails-4-0-to-rails-4-1) Обновление с Rails 4.0 на Rails 4.1
---------------------------------------------------------------------------

### Защита CSRF от внешних тегов `<script>`

Или, "а-а-а-а, почему мои тесты падают!!!?", или "мой виджет `<script>` сломался!!"

Защита от подделки межсайтовых запросов (CSRF) сейчас также покрывает GET запросы с откликами JavaScript. Это предотвращает посторонним сайтам ссылаться на ваши Javascript с помощью тега `<script>` для извлечения конфиденциальных данных.

Это означает, что ваши функциональные и интеграционные тесты, использующие

```ruby
get :index, format: :js
```

теперь будут вызывать защиту CSRF. Переключитесь на

```ruby
xhr :get, :index, format: :js
```

чтобы явно тестировать `XmlHttpRequest`.

Note: Ваши собственные теги `<script>` трактуются как межсайтовые и по умолчанию также блокируются. Если вы действительно хотите загружать JavaScript в тегах `<script>`, теперь нужно явно отключить защиту CSRF для этого экшна.

### Spring

Если хотите использовать Spring в качестве прелоадера своего приложения, вам необходимо:

1. Добавить `gem 'spring', group: :development` в свой `Gemfile`.
2. Установить spring с помощью `bundle install`.
3. Прокачать свои binstub с помощью `bundle exec spring binstub --all`.

NOTE: Пользовательские задачи rake по умолчанию будут запущены в окружении `development`. Если хотите запускать их в других средах, проконсультируйтесь со [Spring README](https://github.com/rails/spring#rake).

### `config/secrets.yml`

Если хотите использовать новое соглашение по хранению секретных данных вашего приложения в `secrets.yml`, вам необходимо:

1. Создать файл `secrets.yml` в директории `config` со следующим содержимым:

    ```yaml
    development:
      secret_key_base:

    test:
      secret_key_base:

    production:
      secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
    ```

2. Использовать существующий `secret_key_base` из инициализатора `secret_token.rb`, чтобы установить переменную среды SECRET_KEY_BASE для всех пользователей, под которыми запускается приложение Rails в среде production. Альтернативно, можно просто скопировать существующий `secret_key_base` из инициализатора `secret_token.rb` в `secrets.yml` в секцию `production`, заменив '<%= ENV["SECRET_KEY_BASE"] %>'.

3. Убрать инициализатор `secret_token.rb`.

4. Использовать `rake secret` для генерации ключей для секций `development` и `test`.

5. Перезапустить сервер.

### Изменения в тестовом хелпере

Если ваш тестовый хелпер содержит вызов `ActiveRecord::Migration.check_pending!`, его можно убрать. Проверка теперь выполняется автоматически при `require 'rails/test_help'`, хотя наличие этой строчки в вашим хелпере ничему не навредит.

### (cookies-serializer) Сериализатор куки

Приложения, созданные до Rails 4.1, используют `Marshal` для сериализации значений куки при хранении подписанных и зашифрованных куки. Если хотите использовать новый, основанный на `JSON`, формат, можно добавить файл инициализатора со следующим содержимым:

```ruby
Rails.application.config.action_dispatch.cookies_serializer = :hybrid
```

Он прозрачно мигрирует ваши существующие куки, сериализованные `Marshal`, в новый формат, основанный на `JSON`.

При использовании сериализатора `:json` или `:hybrid`, следует иметь в виду, что не все объекты Ruby могут быть сериализованы в JSON. Например, объекты `Date` и `Time` будут сериализованы как строки, и у хэшей ключи будут преобразованы в строки.

```ruby
class CookiesController < ApplicationController
  def set_cookie
    cookies.encrypted[:expiration_date] = Date.tomorrow # => Thu, 20 Mar 2014
    redirect_to action: 'read_cookie'
  end

  def read_cookie
    cookies.encrypted[:expiration_date] # => "2014-03-20"
  end
end
```

Советуем хранить в куки только простые данные (строки и числа). Если необходимо хранить сложные объекты. необходимо производить эти преобразования вручную при чтении значений в последующих запросах.

При использовании хранения сессии в куки, все вышесказанное также применяется к хэшам `session` и `flash`.

### Изменилась структура Flash

Ключи сообщении Flash [нормализуются в строки](https://github.com/rails/rails/commit/a668beffd64106a1e1fedb71cc25eaaa11baf0c1). К ним по прежнему можно получить доступ с помощью символа или строки. Итерация по flash будет всегда возвращать строковые ключи:

```ruby
flash["string"] = "a string"
flash[:symbol] = "a symbol"

# Rails < 4.1
flash.keys # => ["string", :symbol]

# Rails >= 4.1
flash.keys # => ["string", "symbol"]
```

Убедитесь, что вы сравниваете ключи сообщений Flash со строками.

### (changes-in-json-handling) Изменения в обработке JSON

Есть несколько важных изменений в обработке JSON в Rails 4.1.

#### Убран MultiJSON

MultiJSON потерял [смысл своего существования](https://github.com/rails/rails/pull/10576) и был убран из Rails.

Если ваше приложение сейчас непосредственно зависит от MultiJSON, у вас несколько вариантов:

1. Добавьте 'multi_json' в свой Gemfile. Отметьте, что это может что-нибудь сломать в будущем

2. Уйти от MultiJSON в пользу использования вместо него `obj.to_json` и `JSON.parse(str)`

WARNING: Нельзя просто заменить `MultiJson.dump` и `MultiJson.load` на `JSON.dump` и `JSON.load`. Эти API гема JSON означают сериализацию и десериализацию произвольных объектов Ruby, и, в основном, [небезопасны](http://www.ruby-doc.org/stdlib-2.2.2/libdoc/json/rdoc/JSON.html#method-i-load).

#### Совместимость с гемом JSON

Исторически у Rails есть несколько проблем совместимости с гемом JSON. Использование `JSON.generate` и `JSON.dump` в приложении Rails могло вызвать неожиданные ошибки.

Rails 4.1 исправил эти проблемы, изолировав свой собственный кодер от гема JSON. API гема JSON будет функционировать, как обычно, но у него не будет доступа к особенностям, специфичным для Rails. Например:

```ruby
class FooBar
  def as_json(options = nil)
    { foo: 'bar' }
  end
end

>> FooBar.new.to_json # => "{\"foo\":\"bar\"}"
>> JSON.generate(FooBar.new, quirks_mode: true) # => "\"#<FooBar:0x007fa80a481610>\""
```

#### Новый кодер JSON

Кодер JSON в Rails 4.1 был переписан, чтобы воспользоваться преимуществами гема JSON. Для большинства приложений это незаметное изменение. Однако, как часть переписывания, следующие особенности были убраны из кодера:

1. Обнаружение кольцевых структур данных
2. Поддержка хука `encode_json`
3. Опция для кодирования объектов `BigDecimal` как числа, вместо строк

Если ваше приложение зависит от одной из этих особенностей, их можно вернуть, добавив гем [`activesupport-json_encoder`](https://github.com/rails/activesupport-json_encoder)
в свой Gemfile.

#### Представление в JSON объектов Time

`#as_json` для объектов с компонентом времени (`Time`, `DateTime`, `ActiveSupport::TimeWithZone`) теперь возвращает по умолчанию с точностью до миллисекунд. Если необходимо сохранить старое поведение без миллисекунд, добавьте следующее в инициализатор:

```
ActiveSupport::JSON::Encoding.time_precision = 0
```

### Использование `return` в инлайн блоках колбэков

Раньше Rails разрешал инлайн блокам колбэков использовать `return` таким образом:

```ruby
class ReadOnlyModel < ActiveRecord::Base
  before_save { return false } # ПЛОХО
end
```

Это поведение никогда явно не поддерживалось. В связи с изменением внутри `ActiveSupport::Callbacks`, оно более недопустимо в Rails 4.1. Использование выражения `return` в инлайн блоке колбэка вызовет `LocalJumpError` при выполнении колбэка.

Использование `return` в инлайн блоке колбэка может быть отрефакторено на вычисление возвращаемого значения:

```ruby
class ReadOnlyModel < ActiveRecord::Base
  before_save { false } # ХОРОШО
end
```

Как вариант, если предпочтителен `return`, рекомендуется явно вызывать метод:

```ruby
class ReadOnlyModel < ActiveRecord::Base
  before_save :before_save_callback # ХОРОШО

  private
    def before_save_callback
      return false
    end
end
```

Это изменение применяется к большинству мест в Rails, где используются колбэки, включая колбэки Active Record и Active Model, а также фильтры в Action
Controller (т.е. `before_action`).

Подробности смотрите в [этом pull request](https://github.com/rails/rails/pull/13271).

### Методы, определенные в фикстурах Active Record

Rails 4.1 вычисляет ERB каждой фикстуры в отдельном контексте, поэтому методы хелпера, определенные в фикстуре, не будут доступны в других фикстурах.

Методы хелпера, используемые в нескольких фикстурах, должны быть определены в модулях, подключаемых в новом `ActiveRecord::FixtureSet.context_class`, в
`test_helper.rb`.

```ruby
module FixtureFileHelpers
  def file_sha(path)
    Digest::SHA2.hexdigest(File.read(Rails.root.join('test/fixtures', path)))
  end
end
ActiveRecord::FixtureSet.context_class.include FixtureFileHelpers
```

### Обеспечение доступных локалей I18n

Сейчас Rails 4.1 устанавливает по умолчанию для опции I18n `enforce_available_locales` `true`. Это означает, что он убедится, что все локали, переданные в него, должны быть объявлены в списке `available_locales`.

Чтобы это отключить (и позволить I18n принимать *любые* локали), добавьте следующую конфигурацию в свое приложение:

```ruby
config.i18n.enforce_available_locales = false
```

Отметьте, что эта опция была добавлена как мера безопасности, чтобы обеспечить, что пользовательская информация не может использоваться как информация о локали, если она не была ранее известна. Следовательно, рекомендуется на отключать эту опцию, если у вас нет весомых причин так делать.

### Мутирующие методы, вызываемые на Relation

У `Relation` больше нет мутирующих методов, таких как `#map!` и `#delete_if`. Преобразовывайте в массив, вызывая `#to_a`, перед использованием этих методов.

Это предназначено для предотвращения странных багов и непонятностей в коде, вызывающем мутирующие методы непосредственно на `Relation`.

```ruby
# Вместо этого
Author.where(name: 'Hank Moody').compact!

# Теперь нужно делать так
authors = Author.where(name: 'Hank Moody').to_a
authors.compact!
```

### (changes-on-default-scopes) Изменения в скоупах по умолчанию

Скоупы по умолчанию больше не переопределяются присоединенными условиями.

В прежних версиях, при определении в модели `default_scope`, он переопределялся присоединенными условиями на то же поле. Теперь он мержится, как и любой другой скоуп.

Раньше:

```ruby
class User < ActiveRecord::Base
  default_scope { where state: 'pending' }
  scope :active, -> { where state: 'active' }
  scope :inactive, -> { where state: 'inactive' }
end

User.all
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending'

User.active
# SELECT "users".* FROM "users" WHERE "users"."state" = 'active'

User.where(state: 'inactive')
# SELECT "users".* FROM "users" WHERE "users"."state" = 'inactive'
```

После:

```ruby
class User < ActiveRecord::Base
  default_scope { where state: 'pending' }
  scope :active, -> { where state: 'active' }
  scope :inactive, -> { where state: 'inactive' }
end

User.all
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending'

User.active
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending' AND "users"."state" = 'active'

User.where(state: 'inactive')
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending' AND "users"."state" = 'inactive'
```

Чтобы получить предыдущее поведение, необходимо явно убрать условие `default_scope` с помощью `unscoped`, `unscope`, `rewhere` или `except`.

```ruby
class User < ActiveRecord::Base
  default_scope { where state: 'pending' }
  scope :active, -> { unscope(where: :state).where(state: 'active') }
  scope :inactive, -> { rewhere state: 'inactive' }
end

User.all
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending'

User.active
# SELECT "users".* FROM "users" WHERE "users"."state" = 'active'

User.inactive
# SELECT "users".* FROM "users" WHERE "users"."state" = 'inactive'
```

### (rendering-content-from-string) Рендеринг содержимого из строки

Rails 4.1 предоставляет опции `:plain`, `:html` и `:body` для `render`. Эти опции теперь являются предпочтительным способом рендеринга основанного на строке содержимого, так как позволяет указать, какой тип содержимого вы хотите отослать в качестве отклика.

* `render :plain` установит тип содержимого `text/plain`
* `render :html` установит тип содержимого `text/html`
* `render :body` *не* установит заголовок типа содержимого.

С точки зрения безопасности, если не ожидается какой-либо разметки в теле отклика, следует использовать `render :plain`, так как большинство браузеров будет экранировать небезопасное содержимое вашего отклика.

Использование `render :text` будет объявлено устаревшим в будущих версиях. Пожалуйста, начинайте использовать более точные опции `:plain`, `:html` и `:body`. Использование `render :text` может вызвать риски безопасности, так как содержимое посылается как `text/html`.

### типы данных json и hstore в PostgreSQL

Rails 4.1 связывает столбцы `json` и `hstore` с Ruby `Hash` со строковыми ключами.
В прежних версиях использовался `HashWithIndifferentAccess`. Это означает, что доступ по символу больше не поддерживается. Это также касается `store_accessors`, основанного на столбцах `json` или `hstore`. Убедитесь, что правильно используете строковые ключи.

### Явное использование блока для `ActiveSupport::Callbacks`

Rails 4.1 теперь ожидает, что будет передан явный блок при вызове `ActiveSupport::Callbacks.set_callback`. Это изменение пришло из `ActiveSupport::Callbacks`, который был существенно переписан для релиза 4.1.

```ruby
# Раньше в Rails 4.0
set_callback :save, :around, ->(r, &block) { stuff; result = block.call; stuff }

# Теперь в Rails 4.1
set_callback :save, :around, ->(r, block) { stuff; result = block.call; stuff }
```

(upgrading-from-rails-3-2-to-rails-4-0) Обновление с Rails 3.2 на Rails 4.0
-------------------------------------

Если версия Rails вашего приложения сейчас старше чем 3.2.x, следует сперва обновиться до Rails 3.2, перед попыткой обновиться до Rails 4.0.

Следующие изменения предназначены для обновления вашего приложения на Rails 4.0.

### HTTP PATCH

Rails 4 теперь использует `PATCH` в качестве основного метода HTTP для обновлений, когда в `config/routes.rb` объявлен RESTful-ресурс. Экшн `update` все еще используется, и запросы `PUT` также будут направлены к экшну `update`. Поэтому, если вы используйте только стандартные RESTful-маршруты, не нужно делать никаких изменений:

```ruby
resources :users
```

```erb
<%= form_for @user do |f| %>
```

```ruby
class UsersController < ApplicationController
  def update
    # No change needed; PATCH will be preferred, and PUT will still work.
  end
end
```

Однако, необходимо сделать изменение, если вы используете `form_for` для обновления ресурса в сочетании с произвольным маршрутом с использованием метода `PUT` HTTP:

```ruby
resources :users, do
  put :update_name, on: :member
end
```

```erb
<%= form_for [ :update_name, @user ] do |f| %>
```

```ruby
class UsersController < ApplicationController
  def update_name
    # Требуется изменение; form_for попытается использовать несуществующий маршрут PATCH.
  end
end
```

Если экшн не используется в публичном API, и можно без проблем изменить метод HTTP, можно обновить маршрут для использования `patch` вместо `put`:

Запросы `PUT` к `/users/:id` в Rails 4 направляются к `update`, как и раньше. Поэтому, если ваше API получит настоящие PUT запросы, они будут работать.
Роутер также направит запросы `PATCH` к `/users/:id` в экшн `update`.

```ruby
resources :users do
  patch :update_name, on: :member
end
```

Если экшн используется в публичном API, и вы не можете изменить используемый метод HTTP, можно обновить форму для использования метода `PUT`:

```erb
<%= form_for [ :update_name, @user ], method: :put do |f| %>
```

Подробнее о PATCH, и почему это изменение было сделано, смотрите [эту публикацию](http://weblog.rubyonrails.org/2012/2/25/edge-rails-patch-is-the-new-primary-http-method-for-updates/) в блоге Rails.

#### Заметка о типах медиа

Корректировка для метода `PATCH` [определяет, что с `PATCH` должен использоваться тип медиа 'diff' ](http://www.rfc-editor.org/errata_search.php?rfc=5789). Один из таких форматов [JSON Patch](http://tools.ietf.org/html/rfc6902). Хотя Rails не поддерживает JSON Patch, такую поддержку легко добавить:

```
# в вашем контроллере
def update
  respond_to do |format|
    format.json do
      # выполнить частичное обновление
      @article.update params[:post]
    end

    format.json_patch do
      # выполнить сложное изменение
    end
  end
end

# В config/initializers/json_patch.rb:
Mime::Type.register 'application/json-patch+json', :json_patch
```

Так как JSON Patch только недавно был добавлен в RFC, пока еще нет множества замечательных библиотек Ruby. Один из имеющихся гемов [hana](https://github.com/tenderlove/hana) от Aaron Patterson, но в нем еще нет полной поддержки нескольких последних изменений в спецификации.

### Gemfile

Rails 4.0 убрал группу `assets` из Gemfile. Вам нужно убрать эту строчку из Gemfile перед обновлением. Также следует обновить файл приложения (`config/application.rb`):

```ruby
# Require the gems listed in Gemfile, including any gems
# you've limited to :test, :development, or :production.
Bundler.require(*Rails.groups)
```

### vendor/plugins

Rails 4.0 больше не поддерживает загрузку плагинов из `vendor/plugins`. Следует переместить любые плагины, извлекая их в гемы и помещая их в Gemfile. Если решаете не делать гемы, можно переместить их, скажем, в `lib/my_plugin/*` и добавить соответствующий инициализатор в `config/initializers/my_plugin.rb`.

### Active Record

* Rails 4.0 убрал identity map из Active Record, из-за [некоторых несоответствий со связями](https://github.com/rails/rails/commit/302c912bf6bcd0fa200d964ec2dc4a44abe328a6). Если вы вручную включали это в своем приложении, нужно убрать соответствующую настройку, так как от нее больше не будет эффекта: `config.active_record.identity_map`.

* Метод `delete` в связях коллекции может получать аргументы `Integer` или `String` в качестве id записей, кроме самих записей, так же, как делает метод `destroy`. Раньше он вызывал `ActiveRecord::AssociationTypeMismatch` для таких аргументов. Начиная с Rails 4.0, `delete` пытается автоматически найти записи, соответствующие переданным id, до их удаления.

* В Rails 4.0, когда переименовывается столбец или таблица, относящиеся к ним индексы также переименовываются. Если у вас есть миграции, переименовывающие индексы – они больше не нужны.

* Rails 4.0 изменил `serialized_attributes` и `attr_readonly` быть только методами класса. Не следует использовать методы экземпляра, так как они устарели. Следует заменить их на методы класса, т.е. `self.serialized_attributes` на `self.class.serialized_attributes`.

* При использовании кодировщика по умолчанию, назначение `nil` сериализованному атрибуту сохранит его в базу данных как `NULL`, вместо пропуска значения `nil` через YAML (`"--- \n...\n"`).

* Rails 4.0 убрал особенность `attr_accessible` и `attr_protected` в пользу. Для более гладкого процесса обновления можно использовать [гем Protected Attributes](https://github.com/rails/protected_attributes).

* Если вы не используете Protected Attributes, можно удалить опции, относящиеся к этому гему, такие как `whitelist_attributes` или `mass_assignment_sanitizer`.

* Rails 4.0 требует, чтобы скоупы использовали вызываемый объект, такой как Proc или lambda:

```ruby
  scope :active, where(active: true)

  # becomes
  scope :active, -> { where active: true }
```

* В Rails 4.0 устарели `ActiveRecord::Fixtures` в пользу `ActiveRecord::FixtureSet`.

* В Rails 4.0 устарел `ActiveRecord::TestCase` в пользу `ActiveSupport::TestCase`.

* В Rails 4.0 устарел API поиска, основанного на хэше. Это означает, что методы, которые раньше принимали "finder options", больше так не делают. Например, `Book.find(:all, conditions: { name: '1984' })` устарел в пользу `Book.where(name: '1984')`

* Все динамические методы, кроме `find_by_...` and `find_by_...!`, устарели. Вот как можно внести изменения:

      * `find_all_by_...`           становится `where(...)`.
      * `find_last_by_...`          становится `where(...).last`.
      * `scoped_by_...`             становится `where(...)`.
      * `find_or_initialize_by_...` становится `find_or_initialize_by(...)`.
      * `find_or_create_by_...`     становится `find_or_create_by(...)`.

* Отметьте, что `where(...)` возвращает relation, а не массив, как старые методы поиска. Если вы ожидаете массив, используйте `where(...).to_a`.

* Эти эквивалентные методы могут выполнять не идентичный SQL с предыдущей реализацией.

* Чтобы включить старые методы поиска, можно использовать [гем activerecord-deprecated_finders](https://github.com/rails/activerecord-deprecated_finders).

* Rails 4.0 изменил соединительную таблицу по умолчанию для связей `has_and_belongs_to_many`, удаляя общий префикс из имени второй таблицы. Любая существующая связь `has_and_belongs_to_many` между моделями с общим префиксом должна быть указана опция `join_table`. Например:

```ruby
CatalogCategory < ActiveRecord::Base
  has_and_belongs_to_many :catalog_products, join_table: 'catalog_categories_catalog_products'
end

CatalogProduct < ActiveRecord::Base
  has_and_belongs_to_many :catalog_categories, join_table: 'catalog_categories_catalog_products'
end
```

* Отметьте, что префикс также принимает во внимание пространства имен, поэтому связи между `Catalog::Category` и `Catalog::Product` или `Catalog::Category` и `CatalogProduct` также необходимо обновить.


### Active Resource

Rails 4.0 извлек Active Resource в отдельный гем. Если вам все еще нужна эта особенность, можете добавить [гем Active Resource](https://github.com/rails/activeresource) в своем Gemfile.

### Active Model

* Rails 4.0 изменил то, как прикрепляются ошибки с помощью `ActiveModel::Validations::ConfirmationValidator`. Теперь, когда не проходят валидации подтверждения, ошибка будет прикреплена к `:#{attribute}_confirmation` вместо `attribute`.

* Rails 4.0 изменил значение по умолчанию для `ActiveModel::Serializers::JSON.include_root_in_json` на `false`. Теперь сериализаторы Active Model и объекты Active Record имеют одинаковое значение по умолчанию. Это означает, что вы можете закомментировать или убрать следующую опцию в файле `config/initializers/wrap_parameters.rb`:

```ruby
# Disable root element in JSON by default.
# ActiveSupport.on_load(:active_record) do
#   self.include_root_in_json = false
# end
```

### Action Pack

* Rails 4.0 представил `ActiveSupport::KeyGenerator`, и использует его, как основу для генерации и проверки подписанных куки (среди прочего). Существующие подписанные куки, сгенерированные с помощью Rails 3.x будут прозрачно обновлены, если вы оставите существующий `secret_token` и добавите новый `secret_key_base`.

```ruby
  # config/initializers/secret_token.rb
  Myapp::Application.config.secret_token = 'existing secret token'
  Myapp::Application.config.secret_key_base = 'new secret key base'
```

Отметьте, что вы должны обождать с установкой `secret_key_base`, пока 100% пользователей на перейдет на Rails 4.x, и вы точно не будете уверены, что не придется откатиться к Rails 3.x. Это так, потому что куки, подписанные на основе нового `secret_key_base` в Rails 4.x, обратно несовместимы с Rails 3.x. Можно спокойно оставить существующий `secret_token`, не устанавливать новый `secret_key_base` и игнорировать предупреждения, пока вы не будете полностью уверены, что обновление полностью завершено.

Если вы полагаетесь на возможность внешних приложений или JavaScript читать подписанные куки сессии вашего приложения Rails (или подписанные куки в целом), вам не следует устанавливать `secret_key_base`, пока вы не избавитесь от этой проблемы.

* Rails 4.0 шифрует содержимое основанной на куки сессии, если был установлен `secret_key_base`. Rails 3.x подписывал, но не шифровал содержимое основанной на куки сессии. Подписанные куки "безопасны", так как проверяется, что они были созданы приложением, и защищены от взлома. Однако, содержимое может быть просмотрено пользователем, и шифрование содержимого устраняет эту заботу без значительного снижения производительности.

Подробнее читайте в [Pull Request #9978](https://github.com/rails/rails/pull/9978) о переходе на подписанные куки сессии.

* Rails 4.0 убрал опцию `ActionController::Base.asset_path`. Используйте особенность файлопровода (assets pipeline).

* В Rails 4.0 устарела опция `ActionController::Base.page_cache_extension`. Используйте вместо нее `ActionController::Base.default_static_extension`.

* Rails 4.0 убрал кэширование страниц и экшнов из Action Pack. Необходимо добавить гем `actionpack-action_caching` для использования `caches_action` и `actionpack-page_caching` для использования `caches_page` в контроллерах.

* Rails 4.0 убрал парсер параметров XML. Следует добавить гем `actionpack-xml_parser`, если вам требуется эта особенность.

* Rails 4.0 изменил набор поиска `layout` по умолчанию с помощью символов или проков, возвращающих nil. Чтобы получить поведение "без макета", возвращайте false вместо nil.

* Rails 4.0 изменил клиент memcached по умолчанию с `memcache-client` на `dalli`. Чтобы обновиться, просто добавьте `gem 'dalli'` в свой `Gemfile`.

* В Rails 4.0 устарели методы `dom_id` и `dom_class` в контроллерах (они нужны только во вьюхах). Вам следует включить модуль `ActionView::RecordIdentifier` в контроллерах, требующих эту особенность.

* В Rails 4.0 устарела опция `:confirm` для хелпера `link_to`. Вместо нее следует полагаться на атрибут data (т.е. `data: { confirm: 'Are you sure?' }`). Это устаревание также затрагивает хелперы, основанные на этом (такие как `link_to_if` или `link_to_unless`).

* Rails 4.0 изменил работу `assert_generates`, `assert_recognizes` и `assert_routing`. Теперь все эти операторы контроля вызывают `Assertion` вместо `ActionController::RoutingError`.

* Rails 4.0 вызывает `ArgumentError`, если определены коллизии в именах маршрутов. Это может быть вызвано как явно определенными именованными маршрутами, либо методом `resources`. Вот два примера, которые вызывают коллизию маршрутов с именем `example_path`:

```ruby
  get 'one' => 'test#example', as: :example
  get 'two' => 'test#example', as: :example
```

```ruby
  resources :examples
  get 'clashing/:id' => 'test#example', as: :example
```

В первом случае можно просто избежать использование одинакового имени для нескольких маршрутов. Во втором следует использовать опции `only` или `except`, представленные методом `resources`, чтобы ограничить создаваемые маршруты, о чем подробно описано в [Руководстве по роутингу](/rails-routing#restricting-the-routes-created).

* Rails 4.0 также изменил способ отрисовки маршрутов с символами unicode. Теперь можно непосредственно отрисовывать символы unicode character. Если вы уже отрисовываете такие маршруты, их нужно изменить, например:

```ruby
get Rack::Utils.escape('こんにちは'), controller: 'welcome', action: 'index'
```

станет

```ruby
get 'こんにちは', controller: 'welcome', action: 'index'
```

* Rails 4.0 требует, чтобы маршруты, использующие `match` указывали метод запроса. Например:

```ruby
  # Rails 3.x
  match '/' => 'root#index'

  # станет
  match '/' => 'root#index', via: :get

  # или
  get '/' => 'root#index'
```

* В Rails 4.0 убрана промежуточная программа `ActionDispatch::BestStandardsSupport`, `<!DOCTYPE html>` уже включает режим стандартов в соответствии с http://msdn.microsoft.com/en-us/library/jj676915(v=vs.85).aspx, а заголовок ChromeFrame был перемещен в `config.action_dispatch.default_headers`.

Помните, что вы также должны убрать все упоминания промежуточной программы из кода своего приложения, например:

```ruby
# Вызовет исключение
config.middleware.insert_before(Rack::Lock, ActionDispatch::BestStandardsSupport)
```

Также найдите в своих настройках сред `config.action_dispatch.best_standards_support`, и уберите эту строчку, если она есть.

* В Rails 4.0 при прекомпиляции ресурсов не будут больше автоматически копироваться не-JS/CSS ресурсы из `vendor/assets` и `lib/assets`. Разработчики приложений Rails и engine-ов должны поместить эти ресурсы в `app/assets` или настроить `config.assets.precompile`.

* В Rails 4.0 вызывается `ActionController::UnknownFormat`, когда экшн не обрабатывает формат запроса. По умолчанию исключение обрабатывается, откликаясь с помощью 406 Not Acceptable, но теперь это можно переопределить. В Rails 3 всегда возвращался 406 Not Acceptable. Без возможности переопределения.

* В Rails 4.0 вызывается характерное исключение `ActionDispatch::ParamsParser::ParseError`, когда `ParamsParser` не сможет спарсить параметры запроса. Вам нужно ловить это исключение, вместо низкоуровневого `MultiJson::DecodeError`, например.

* В Rails 4.0 `SCRIPT_NAME` правильно вкладывается, когда engine монтируется в приложении, находящемся на префиксе URL. Больше не нужно устанавливать `default_url_options[:script_name]`, чтобы работать с переписанными префиксами URL.

* В Rails 4.0 устарел `ActionController::Integration` в пользу `ActionDispatch::Integration`.
* В Rails 4.0 устарел `ActionController::IntegrationTest` в пользу `ActionDispatch::IntegrationTest`.
* В Rails 4.0 устарел `ActionController::PerformanceTest` в пользу `ActionDispatch::PerformanceTest`.
* В Rails 4.0 устарел `ActionController::AbstractRequest` в пользу `ActionDispatch::Request`.
* В Rails 4.0 устарел `ActionController::Request` в пользу `ActionDispatch::Request`.
* В Rails 4.0 устарел `ActionController::AbstractResponse` в пользу `ActionDispatch::Response`.
* В Rails 4.0 устарел `ActionController::Response` в пользу `ActionDispatch::Response`.
* В Rails 4.0 устарел `ActionController::Routing` в пользу `ActionDispatch::Routing`.

### Active Support

Rails 4.0 убрал псевдоним `j` для `ERB::Util#json_escape`, так как `j` уже используется для `ActionView::Helpers::JavaScriptHelper#escape_javascript`.

#### Cache

Метод кэширования изменился между Rails 3.x и 4.0. Вы должны [изменить пространство имен](/caching-with-rails-an-overview#activesupport-cache-store) и развертывать с холодным кэшем.

### Порядок загрузки хелперов

В Rails 4.0 изменился порядок, в котором загружались хелперы из более чем одной директории. Ранее они собирались, а затем сортировались по алфавиту. После обновления на Rails 4.0, хелперы будут сохранять порядок загружаемых директорий и будут сортироваться по алфавиту только в пределах каждой директории. Если вы явно не используете параметр `helpers_path`, Это изменение повлияет только на способ загрузки хелперов из engine-ов. Если вы полагаетесь на порядок загрузки, следует проверить, что после обновления доступны правильные методы. Если хотите изменить порядок, в котором загружаются engine, Можно использовать метод `config.railties_order=`.

### Active Record Observer и Action Controller Sweeper

`ActiveRecord::Observer` и `ActionController::Caching::Sweeper` были извлечены в гем `rails-observers`. Следует добавить гем `rails-observers`, если вам нужны эти особенности.

### sprockets-rails

* `assets:precompile:primary` и `assets:precompile:all` были убраны. Используйте вместо них `assets:precompile`.
* Опция `config.assets.compress` должна быть изменена на `config.assets.js_compressor`, например, так:

```ruby
config.assets.js_compressor = :uglifier
```

### sass-rails

* `asset_url` с двумя аргументами устарел. Например: `asset-url("rails.png", image)` стал `asset-url("rails.png")`


Обновление с Rails 3.1 на Rails 3.2
-------------------------------------

Если версия Rails вашего приложения сейчас старше чем 3.1.x, следует сперва обновиться до Rails 3.1, перед попыткой обновиться до Rails 3.2.

Следующие изменения предназначены для обновления вашего приложения на последнюю версию 3.2.x Rails.

### Gemfile

Сделайте следующие изменения в своем `Gemfile`.

```ruby
gem 'rails', '3.2.21'

group :assets do
  gem 'sass-rails',   '~> 3.2.6'
  gem 'coffee-rails', '~> 3.2.2'
  gem 'uglifier',     '>= 1.0.3'
end
```

### config/environments/development.rb

Имеется ряд новых конфигурационных настроек, которые следует добавить в среде development:

```ruby
# Raise exception on mass assignment protection for Active Record models
config.active_record.mass_assignment_sanitizer = :strict

# Log the query plan for queries taking more than this (works
# with SQLite, MySQL, and PostgreSQL)
config.active_record.auto_explain_threshold_in_seconds = 0.5
```

### config/environments/test.rb

Также должна быть добавлена конфигурационная настройка `mass_assignment_sanitizer` в `config/environments/test.rb`:

```ruby
# Raise exception on mass assignment protection for Active Record models
config.active_record.mass_assignment_sanitizer = :strict
```

### vendor/plugins

В Rails 3.2 устаревает `vendor/plugins`, а в Rails 4.0 будет убрана полностью. Хотя это и не требуется строго при обновлении на Rails 3.2, можно начать перемещать любые плагины, извлекая их в гемы и помещая их в Gemfile. Если решаете не делать гемы, можно переместить их, скажем, в `lib/my_plugin/*` и добавить соответствующий инициализатор в `config/initializers/my_plugin.rb`.

### Active Record

Опция `:dependent => :restrict` была убрана из `belongs_to`. Если хотите предотвратить удаление объекта, если имеются какие-либо связанные объекты, можно установить `:dependent => :destroy` и возвращать `false` после проверки существования связи из любого кколбэка на destroy связанного объекта.

Обновление с Rails 3.0 на Rails 3.1
-------------------------------------

Если версия Rails вашего приложения сейчас старше чем 3.0.x, следует сперва обновиться до Rails 3.0, перед попыткой обновиться до Rails 3.1.

Следующие изменения предназначены для обновления вашего приложения на Rails 3.1.12, последнюю версию 3.1.x Rails.

### Gemfile

Сделайте следующие изменения в своем `Gemfile`.

```ruby
gem 'rails', '3.1.12'
gem 'mysql2'

# Needed for the new asset pipeline
group :assets do
  gem 'sass-rails',   '~> 3.1.7'
  gem 'coffee-rails', '~> 3.1.1'
  gem 'uglifier',     '>= 1.0.3'
end

# jQuery is the default JavaScript library in Rails 3.1
gem 'jquery-rails'
```

### config/application.rb

Файлопровод (asset pipeline) требует следующих добавлений:

```ruby
config.assets.enabled = true
config.assets.version = '1.0'
```

Если ваше приложение использует маршрут "/assets" для ресурса, можно изменить префикс, используемый для файлов, чтобы избежать конфликтов:

```ruby
# Defaults to '/assets'
config.assets.prefix = '/asset-files'
```

### config/environments/development.rb

Уберите настройку для RJS `config.action_view.debug_rjs = true`.

Добавьте эти настройки, если вы включили файлопровод:

```ruby
# Do not compress assets
config.assets.compress = false

# Expands the lines which load the assets
config.assets.debug = true
```

### config/environments/production.rb

Снова, большая часть изменений относится к файлопроводу. Подробнее о них можно прочитать в руководстве по [Asset Pipeline](/asset-pipeline).

```ruby
# Compress JavaScripts and CSS
config.assets.compress = true

# Don't fallback to assets pipeline if a precompiled asset is missed
config.assets.compile = false

# Generate digests for assets URLs
config.assets.digest = true

# Defaults to Rails.root.join("public/assets")
# config.assets.manifest = YOUR_PATH

# Precompile additional assets (application.js, application.css, and all non-JS/CSS are already added)
# config.assets.precompile += %w( admin.js admin.css )

# Force all access to the app over SSL, use Strict-Transport-Security, and use secure cookies.
# config.force_ssl = true
```

### config/environments/test.rb

Можно увеличить производительность тестов, добавив следующее в среде test:

```ruby
# Configure static asset server for tests with Cache-Control for performance
config.public_file_server.enabled = true
config.public_file_server.headers = {
  'Cache-Control' => 'public, max-age=3600'
}
```

### config/initializers/wrap_parameters.rb

Добавьте эти файлы со следующим содержимым, если хотите оборачивать параметры во вложенный хэш. Для новых приложений это включено по умолчанию.

```ruby
# Be sure to restart your server when you modify this file.
# This file contains settings for ActionController::ParamsWrapper which
# is enabled by default.

# Enable parameter wrapping for JSON. You can disable this by setting :format to an empty array.
ActiveSupport.on_load(:action_controller) do
  wrap_parameters format: [:json]
end

# Disable root element in JSON by default.
ActiveSupport.on_load(:active_record) do
  self.include_root_in_json = false
end
```

### config/initializers/session_store.rb

Необходимо изменить ключ сессии на другой, или удалить все сессии:

```ruby
# in config/initializers/session_store.rb
AppName::Application.config.session_store :cookie_store, key: 'SOMETHINGNEW'
```

или

```bash
$ bin/rake db:sessions:clear
```

### Убрать опции :cache и :concat в ресурсных хелперах во вьюхах

* Вместе с Asset Pipeline опции :cache и :concat больше не используются, удалите их из вьюх.
