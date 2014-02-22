Заметки о релизе Ruby on Rails 4.1
===============================

Ключевые новинки в Rails 4.1:

* Spring прелоадер
* `config/secrets.yml`
* Action Pack Variants (шаблоны, для разных устройств)
* Предпросмотр писем Action Mailer

Эти заметки о релизе покрывают только основные обновления. Чтобы узнать о различных багфиксах и изменениях, обратитесь к логам изменений или к
[списку комитов](https://github.com/rails/rails/commits/master) в главном репозитории Rails на GitHub.


--------------------------------------------------------------------------------

Обновление до Rails 4.1
----------------------

Если вы обновляете существующее приложение, было бы хорошо иметь перед этим покрытие тестами.
Для начала, вы должны обновиться до Rails 4.0 и убедиться что ваше приложение по-прежнему работает,
как ожидалось, прежде чем пытаться обновиться до Rails 4.1.
Список вещей, которые нужно выполнить для обновления доступен в руководстве
[Обновление Rails](upgrading_ruby_on_rails.html#upgrading-from-rails-4-0-to-rails-4-1).


Основные изменения
--------------

### Spring Application Preloader

Spring является прелоадером для Rails приложений. Он увеличивает скорость разработки, храня
ваше приложение запущенным в бэкграунде, поэтому при прогоне тестов, rake тасков или миграций, теперь загружать приложение каждый раз больше нет необходимости.

Новое Rails 4.1 приложение будет по умолчанию идти с "springified" бинстабами. Это означает, что
`bin/rails` и `bin/rake`, будут автоматически использовать преимущества предзагруженной среды spring.

**Запуск rake задач:**

```
bin/rake test:models
```

**Запуск Rails комманд:**

```
bin/rails console
```

**Spring интроспекция:**

```
$ bin/spring status
Spring is running:

 1182 spring server | my_app | started 29 mins ago
 3656 spring app    | my_app | started 23 secs ago | test mode
 3746 spring app    | my_app | started 10 secs ago | development mode
```

Взгляните на
[Spring README](https://github.com/jonleighton/spring/blob/master/README.md),
чтобы увидеть все возможности.

Взгляните на руководство по [Обновлению Rails](upgrading_ruby_on_rails.html#spring)
- как мигрировать существующее приложение, чтобы использовать данную возможность.

### `config/secrets.yml`

Rails 4.1 генерирует новый файл `secrets.yml` в директории `config`. По умолчанию,
этот файл содержит `secret_key_base` приложения, но он так же может использоваться
для хранения других секретных данных, таких как ключи доступа к внешним API.

Секретные данные, добавляемые в этот файл, будут доступны  через `Rails.application.secrets`.
К примеру, `config/secrets.yml`:

```yaml
development:
  secret_key_base: 3b7cd727ee24e8444053437c36cc66c3
  some_api_key: SOMEKEY
```

`Rails.application.secrets.some_api_key` вернёт `SOMEKEY` в development окружении.

Взгляните на руководство по [Обновлению Rails](upgrading_ruby_on_rails.html#config-secrets-yml)
- как мигрировать существующее приложение, чтобы использовать данную возможность.

### Action Pack Variants

Мы часто хотим рендерить разные типы шаблонов HTML/JSON/XML - для телефонов,
планшетов и десктопных компьютеров. С помощью Variants - это легко.

Запрос Variants - это специальный формат запроса, например `:tablet`,
`:phone`, или `:desktop`.

Вы можете установить вариант шаблона в `before_action`:

```ruby
request.variant = :tablet if request.user_agent =~ /iPad/
```

Респонс варианта шаблона в экшене, такой же как респонс у различных форматов

```ruby
respond_to do |format|
  format.html do |html|
    html.tablet # renders app/views/projects/show.html+tablet.erb
    html.phone { extra_setup; render ... }
  end
end
```

Создайте отдельные шаблоны для каждого формата и варианта шаблона:

```
app/views/projects/show.html.erb
app/views/projects/show.html+tablet.erb
app/views/projects/show.html+phone.erb
```

Вы так же можете упростить определение вариантов шаблонов используя строчный синтаксис:

```ruby
respond_to do |format|
  format.js         { render "trash" }
  format.html.phone { redirect_to progress_path }
  format.html.none  { render "trash" }
end
```

### Предпросмотр писем Action Mailer

Предпросмотр писем Action Mailer, это возможность увидеть как будет выглядеть email,
посетив специальный URL адрес, который покажет ваше письмо.

Вы реализуете класс, методы которого возвращают email объект,
который необходимо проверить:

```ruby
class NotifierPreview < ActionMailer::Preview
  def welcome
    Notifier.welcome(User.first)
  end
end
```

Предпросмотр данного письма доступен по адресу http://localhost:3000/rails/mailers/notifier/welcome,
так же можно увидеть полный список писем - http://localhost:3000/rails/mailers.

По умолчанию, эти превью-классы располагаются в `test/mailers/previews`.
Директорию можно легко изменить используя `preview_path` опцию.

Посмотреть
[документацию](http://api.rubyonrails.org/v4.1.0/classes/ActionMailer/Base.html)
для детального изучения.

### Active Record enum поля

Объявляйте в базе данных enum поле, в котором каждое значение принадлежит цифре,
но может быть запрошено по имени

```ruby
class Conversation < ActiveRecord::Base
  enum status: [ :active, :archived ]
end

conversation.archived!
conversation.active? # => false
conversation.status  # => "archived"

Conversation.archived # => Связь для всех архивированных бесед
```

Посмотреть
[документацию](http://api.rubyonrails.org/v4.1.0/classes/ActiveRecord/Enum.html)
для детального изучения.

### Message Verifiers

Message verifiers могут быть использованы для генерации и верификации подписанных сообщений.
Это может быть полезным для безопасной передачи деликатных данных, таких как токены запомни-меня и др.

Метод `Rails.application.message_verifier` возвращает верификационное сообщение, которое
подтверждено ключём, полученным из secret_key_base и именем верификационного сообщения:

```ruby
signed_token = Rails.application.message_verifier(:remember_me).generate(token)
Rails.application.message_verifier(:remember_me).verify(signed_token) # => token

Rails.application.message_verifier(:remember_me).verify(tampered_token)
# raises ActiveSupport::MessageVerifier::InvalidSignature
```

### Module#concerning

Естественно с низким разделением логики,
обычно быстрый способ разделить ответственность используя класс является:

```ruby
class Todo < ActiveRecord::Base
  concerning :EventTracking do
    included do
      has_many :events
    end

    def latest_event
      ...
    end

    private
      def some_internal_method
        ...
      end
  end
end
```

Этот пример является определением модуля `EventTracking` внутри класса,
расширяя `ActiveSupport::Concern`, и смешиваясь с
`Todo` классом.

Посмотреть
[документацию](http://api.rubyonrails.org/v4.1.0/classes/Module/Concerning.html)
для детального изучения и способох использования.

### CSRF защита от `<script>` тегов

Защитой от подделки межсайтовых запросов (CSRF) сейчас является GET запрос с
JavaScript ответом. Это предотвращает связывание вашего Javascript URL с сторонними сайтами
и попыток запуска его для извлечения конфиденциальных данных.

Это означает, что каждый из ваших тестов который использует `.js` URLы теперь будет
провален с CSRF защитой, если не используется `xhr`. Обновите ваши тесты, чтобы быть уверенными в
XmlHttp запросах. Вместо `post :create, format: :js`, переключитесь на явное
`xhr :post, :create, format: :js`.

Railties
--------

Пожалуйста, обратитесь к
[Changelog](https://github.com/rails/rails/blob/4-1-stable/railties/CHANGELOG.md)
для просмотра всех изменений.

### Удалено

* Удалён rake таск `update:application_controller`.

* Удалён устаревший `Rails.application.railties.engines`.

* Удалён устаревший `threadsafe!` из Rails конфига.

* Удалён устаревший метод `ActiveRecord::Generators::ActiveModel#update_attributes` взамен
  на `ActiveRecord::Generators::ActiveModel#update`.

* Удалёна устаревшая `config.whiny_nils` опция.

* Удалёны устаревшие rake таски для запуска тестов: `rake test:uncommitted` and
  `rake test:recent`.

### Значемые изменения

* [Spring прелоадер](https://github.com/jonleighton/spring) теперь устанавливается по умолчанию
  для новых приложений. Он использует группу development в Gemfile, поэтому не будет установлен в
  продакшене. ([Pull Request](https://github.com/rails/rails/pull/12958))

* `BACKTRACE` переменная окружения, которая показывает нефильтрованные бэктрейсы для проваленных тестов.
  ([Commit](https://github.com/rails/rails/commit/84eac5dab8b0fe9ee20b51250e52ad7bfea36553))

* Возможность конфигурирования `MiddlewareStack#unshift`.
  ([Pull Request](https://github.com/rails/rails/pull/12479))

* Добавлен метод `Application#message_verifier` которы возвращает верификационное
  сообщение. ([Pull Request](https://github.com/rails/rails/pull/12995))

Action Pack
-----------

Пожалуйста обратитесь к
[Changelog](https://github.com/rails/rails/blob/4-1-stable/actionpack/CHANGELOG.md)
для просмотра всех изменений.

### Удалено

* Удалён устаревшний Rails fallback для интеграционных тестов, используйте `ActionDispatch.test_app`.

* Удалён устаревший метод `page_cache_extension`.

* Удалён устаревший `ActionController::RecordIdentifier`, используйте
  `ActionView::RecordIdentifier`.

* Удалены устаревшие константы из Action Controller:

  | Удалено                            | Приемник                        |
  |:-----------------------------------|:--------------------------------|
  | ActionController::AbstractRequest  | ActionDispatch::Request         |
  | ActionController::Request          | ActionDispatch::Request         |
  | ActionController::AbstractResponse | ActionDispatch::Response        |
  | ActionController::Response         | ActionDispatch::Response        |
  | ActionController::Routing          | ActionDispatch::Routing         |
  | ActionController::Integration      | ActionDispatch::Integration     |
  | ActionController::IntegrationTest  | ActionDispatch::IntegrationTest |

### Значемые изменения

* `protect_from_forgery` предотвращает от CSRF атак, проводимых через `<script>` теги.
  Обновите ваши тесты и используйте `xhr :get, :foo, format: :js` вместо
  `get :foo, format: :js`.
  ([Pull Request](https://github.com/rails/rails/pull/13345))

* `#url_for` принимает хэш с опциями внутри массива.
  ([Pull Request](https://github.com/rails/rails/pull/9599))

* Добавлен метод `session#fetch` который ведёт себя аналогично
  [Hash#fetch](http://www.ruby-doc.org/core-1.9.3/Hash.html#method-i-fetch),
  с исключением что возвращаемое значение всегда сохраняется в сессии.
  ([Pull Request](https://github.com/rails/rails/pull/12692))

* Полностью отделён Action View от Action Pack.
  ([Pull Request](https://github.com/rails/rails/pull/11032))

Action Mailer
-------------

Пожалуйста обратитесь к
[Changelog](https://github.com/rails/rails/blob/4-1-stable/actionmailer/CHANGELOG.md)
для просмотра всех изменений.

### Значемые изменения

* Инструмент создания Action Mailer сообщений. Время потраченное на генерацию сообщения
  записывается в лог. ([Pull Request](https://github.com/rails/rails/pull/12556))

Active Record
-------------

Пожалуйста обратитесь к
[Changelog](https://github.com/rails/rails/blob/4-1-stable/activerecord/CHANGELOG.md)
для просмотра всех изменений.

### Удалено

* Удалена устаревшая возможность nil-passing, следующим методам `SchemaCache`:
  `primary_keys`, `tables`, `columns` и `columns_hash`.

* Удалён устаревший блок фильтр из `ActiveRecord::Migrator#migrate`.

* Удалён устаревший конструктор строк из `ActiveRecord::Migrator`.

* Удалёно устаревшее использование `scope` без передачи вызываемого объекта.

* Удалён устаревший метод `transaction_joinable=` взамен на использование `begin_transaction`
  с опцией `d:joinable`.

* Удалён устаревший метод `decrement_open_transactions`.

* Удалён устаревший метод `increment_open_transactions`.

* Удалён устаревший метод `PostgreSQLAdapter#outside_transaction?`.
  Вместо него вы можете использовать `#transaction_open?`.

* Удалён устаревший метод `ActiveRecord::Fixtures.find_table_name` в пользу
  `ActiveRecord::Fixtures.default_fixture_model_name`.

* Удален устаревший метод `columns_for_remove` из `SchemaStatements`.

* Удалён `SchemaStatements#distinct`.

* Перемещён устаревший `ActiveRecord::TestCase` в Rails test
  suite. Данный класс больше не публичный и используется только для внутреннего тестирования Rails.

* Удалена поддержка опции `:restrict` для `:dependent`
  в ассоциациях.

* Удалена поддержка `:delete_sql`, `:insert_sql`, `:finder_sql`
  и `:counter_sql` в ассоциациях.

* Удален устаревший метод `type_cast_code` из ActiveRecord::ConnectionAdapters::Column.

* Удален устаревший метод `ActiveRecord::Base#connection`.
  Убедитесь, что вы обращяетесь к соединению через класс.

* Удалены устаревшие предупреждения `auto_explain_threshold_in_seconds`.

* Удалена устаревшая опция `:distinct` из `Relation#count`.

* Удалены устаревшие методы  `partial_updates`, `partial_updates?` и
  `partial_updates=`.

* Удален устаревший метод `scoped`.

* Удален устаревший метод `default_scopes?`.

* Удалено неявное соединение связей, которое были объявлено устаревшим в 4.0.

* Удален из зависимостей гем `activerecord-deprecated_finders`.

* `implicit_readonly` больше не используется. Пожалуйста, используйте `readonly` метод
  для явной пометки записи как
  `readonly`. ([Pull Request](https://github.com/rails/rails/pull/10769))

### Устарело

* Устарел неиспользуемый метод `quoted_locking_column`.

* Устарело делегирование методов класса Array для ассоциаций.
  Для их использования, вызовите`#to_a` на ассоциации, чтобы получить доступ
  к массиву который будет использоваться.
  ([Pull Request](https://github.com/rails/rails/pull/12129))

* Устарел метод `ConnectionAdapters::SchemaStatements#distinct`,
  так как больше не используется внутри Rails. ([Pull Request](https://github.com/rails/rails/pull/10556))

### Значимые изменения

* Добавлен метод `ActiveRecord::Base.to_param` для удобного создания "красивых" URLов, используя
  атрибуты или методы модели.
  ([Pull Request](https://github.com/rails/rails/pull/12891))

* Добавлена опция `ActiveRecord::Base.no_touching`, которая позволяет игнорировать "touch"
  на модели. ([Pull Request](https://github.com/rails/rails/pull/12772))

* Унификаця преобразования boolean типов для `MysqlAdapter` и `Mysql2Adapter`.
  `type_cast` вернёт `1` для `true` и `0` для `false`. ([Pull Request](https://github.com/rails/rails/pull/12425))

* `.unscope` теперь удаляет условия выборки, опередлённых в скоупе по умолчанию
  `default_scope`. ([Commit](https://github.com/rails/rails/commit/94924dc32baf78f13e289172534c2e71c9c8cade))

* Добавлен метод `ActiveRecord::QueryMethods#rewhere`, который перезаписывает существующее условие where,
  использовавшееся ранее в цепочке запросов.
  ([Commit](https://github.com/rails/rails/commit/f950b2699f97749ef706c6939a84dfc85f0b05f2))

* Расширен метод `ActiveRecord::Base#cache_key`, который теперь принимает опциональный список timestamp
  аттрибутов, из которых самое больше будет использоваться. ([Commit](https://github.com/rails/rails/commit/e94e97ca796c0759d8fcb8f946a3bbc60252d329))

* Добавлен `ActiveRecord::Base#enum` для описания enum аттрибутов, в которых значения
  принадлежат цифрам в базе данных, но могут быть запрошены используя
  имя. ([Commit](https://github.com/rails/rails/commit/db41eb8a6ea88b854bf5cd11070ea4245e1639c5))

* Type cast json values on write, so that the value is consistent with reading
  from the database. ([Pull Request](https://github.com/rails/rails/pull/12643))

* Type cast hstore values on write, so that the value is consistent
  with reading from the database. ([Commit](https://github.com/rails/rails/commit/5ac2341fab689344991b2a4817bd2bc8b3edac9d))

* Стало возможным использование `next_migration_number` для разных
  генераторов. ([Pull Request](https://github.com/rails/rails/pull/12407))

* Вызов `update_attributes` теперь бросает исключение `ArgumentError`, когда
  принимает `nil` аргумент. Более конкретно - будет ошибка, если передаваемый аргумент
  не отвечает на `stringify_keys`. ([Pull Request](https://github.com/rails/rails/pull/9860))

* `CollectionAssociation#first`/`#last` (например `has_many`) ограничевает
  результат запроса оператором `LIMIT` в запросе на выборку, вместо загрузки полной коллекции.
  ([Pull Request](https://github.com/rails/rails/pull/12137))

* Метод `inspect` вызываемый на моделях Active Record не инициализирует нового
  подключения. Это означает что вызов `inspect` больше не вызывает исключения, когда база данных
  отсутствует. ([Pull Request](https://github.com/rails/rails/pull/11014))

* Удалено ограничение столбцов для `count`, теперь будет брошено исключение если SQL
  не валидный. ([Pull Request](https://github.com/rails/rails/pull/10710))

* Rails теперь автоматически определяет обратные ассоциации. Если вы не установили опцию
  `:inverse_of`, Active Record самостоятельно определит обратную ассоциацию, основываясь на эвристике.
  ([Pull Request](https://github.com/rails/rails/pull/10886))

* Оперирование алиасами атрибутов в ActiveRecord::Relation. Когда вы используете символы,
  ActiveRecord теперь приводит имена алиасов аттрибутов к фактическим именам столбцов
  используемых в базе данных. ([Pull Request](https://github.com/rails/rails/pull/7839))

* Шаблоны ERB в фикстурах больше не выполняются в контексте главного объекта.
  Методы хелперов использующиеся в нескольких фикстурах должны объявляться в модулях
  включённых в `ActiveRecord::FixtureSet.context_class`. ([Pull Request](https://github.com/rails/rails/pull/13022))

Active Model
------------

Пожалуйста обратитесь к
[Changelog](https://github.com/rails/rails/blob/4-1-stable/activemodel/CHANGELOG.md)
для просмотра всех изменений.

### Устарело

* Устарел `Validator#setup`. Теперь необходимые настройки устанавливаются в конструкторе валидатора.
 ([Commit](https://github.com/rails/rails/commit/7d84c3a2f7ede0e8d04540e9c0640de7378e9b3a))

### Значимые изменения

*  Добавлены новые API методы `reset_changes` и `changes_applied` к
  `ActiveModel::Dirty` которые контролируют изменения состояния.


Active Support
--------------

Пожалуйста обратитесь к
[Changelog](https://github.com/rails/rails/blob/4-1-stable/activesupport/CHANGELOG.md)
для просмотра всех изменений.


### Удалено

* Удалена зависимость `MultiJSON`. Теперь, `ActiveSupport::JSON.decode`
  больше не принимает хеш опций для `MultiJSON`. ([Pull Request](https://github.com/rails/rails/pull/10576) / [Подробнее](upgrading_ruby_on_rails.html#changes-in-json-handling))

* Удалена поддержка `encode_json`, используемого для преобразования кастомных объектов в
  JSON. Данный функционал выделен в гем [activesupport-json_encoder](https://github.com/rails/activesupport-json_encoder).
  ([Связанный Pull Request](https://github.com/rails/rails/pull/12183) /
  [Подробнее](upgrading_ruby_on_rails.html#changes-in-json-handling))

* Удалёно без замены `ActiveSupport::JSON::Variable`.

* Удалёно устаревшее расширение ядра `String#encoding_aware?` (`core_ext/string/encoding`).

* Удалён устаревший метод `Module#local_constant_names` взамен на `Module#local_constants`.

* Удалён устаревший метод `DateTime.local_offset` взамен на `DateTime.civil_from_format`.

* Удалёно устаревшее расширение ядра `Logger` (`core_ext/logger.rb`).

* Удалены устаревшие методы `Time#time_with_datetime_fallback`, `Time#utc_time` и
  `Time#local_time` взамен на `Time#utc` и `Time#local`.

* Удалён устаревший метод `Hash#diff` без замены.

* Удалён устаревший метод `Date#to_time_in_current_zone` взамен на `Date#in_time_zone`.

* Удалён устаревший метод `Proc#bind` без замены.

* Удалены устаревшие методы `Array#uniq_by` и `Array#uniq_by!`, используйте нативные методы класса
  `Array#uniq` и `Array#uniq!`.

* Удалён устаревший `ActiveSupport::BasicObject`, используйте
  `ActiveSupport::ProxyObject`.

* Удалён устаревший `BufferedLogger`, используйте `ActiveSupport::Logger` взамен.

* Удалены устаревшие методы `assert_present` и `assert_blank`, используйте `assert
  object.blank?` и `assert object.present?` взамен.

### Устарело

* Устарело `Numeric#{ago,until,since,from_now}`, пользователь должен явно
  преобразовывать значение в AS::Duration, например. `5.ago` => `5.seconds.ago`
  ([Pull Request](https://github.com/rails/rails/pull/12389))

* Устарело имя подключаемой директории `active_support/core_ext/object/to_json`. Подключайте
  `active_support/core_ext/object/json` взамен. ([Pull Request](https://github.com/rails/rails/pull/12203))

* Устарело `ActiveSupport::JSON::Encoding::CircularReferenceError`. Данный функционал
  был выделен в гем [activesupport-json_encoder](https://github.com/rails/activesupport-json_encoder).
  ([Pull Request](https://github.com/rails/rails/pull/12785) /
  [Подробности](upgrading_ruby_on_rails.html#changes-in-json-handling))

* Устарела опция `ActiveSupport.encode_big_decimal_as_string`. Данный функционал
  был выделен в гем [activesupport-json_encoder](https://github.com/rails/activesupport-json_encoder).
  ([Pull Request](https://github.com/rails/rails/pull/13060) /
  [Подробности](upgrading_ruby_on_rails.html#changes-in-json-handling))

### Значимые изменения

* `ActiveSupport`'s JSON encoder был переписан, для того, чтобы воспользоваться
  гемом JSON, а не создавать свой велосипед.
  ([Pull Request](https://github.com/rails/rails/pull/12183) /
  [Подробности](upgrading_ruby_on_rails.html#changes-in-json-handling))

* Улучшена совместимость с гемом JSON.
  ([Pull Request](https://github.com/rails/rails/pull/12862) /
  [Подробности](upgrading_ruby_on_rails.html#changes-in-json-handling))

* Добалены методы `ActiveSupport::Testing::TimeHelpers#travel` и `#travel_to`. Которые
  изменяют текущее время на промежуток который вы укажите, используя стаб
  `Time.now` и
  `Date.today`. ([Pull Request](https://github.com/rails/rails/pull/12824))

* Добавлен метод `Numeric#in_milliseconds`, можно использовать как `1.hour.in_milliseconds`.
  Теперь вы можете скармливать такие значения в JavaScript функции 
  `getTime()`. ([Commit](https://github.com/rails/rails/commit/423249504a2b468d7a273cbe6accf4f21cb0e643))

* Добавлены методы `Date#middle_of_day`, `DateTime#middle_of_day` и `Time#middle_of_day`.
  Так же добавлены алиасы `midday`, `noon`, `at_midday`, `at_noon` и
  `at_middle_of_day`. ([Pull Request](https://github.com/rails/rails/pull/10879))

* Добавлен метод `String#remove(pattern)` как сокращение для
  `String#gsub(pattern,'')`. ([Commit](https://github.com/rails/rails/commit/5da23a3f921f0a4a3139495d2779ab0d3bd4cb5f))

* Удалёно [словопреобразование](http://api.rubyonrails.org/classes/ActiveSupport/Inflector.html) 'cow' => 'kine'
  из конфига. ([Commit](https://github.com/rails/rails/commit/c300dca9963bda78b8f358dbcb59cabcdc5e1dc9))

Заслуги
-------

Взгляните
[на полный список контрибьюторов Rails](http://contributors.rubyonrails.org/), на людей
которые потратили много часов, сделав Rails стабильнее и надёжнее.
Спасибо им всем.