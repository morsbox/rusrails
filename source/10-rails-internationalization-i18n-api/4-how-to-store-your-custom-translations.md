# Как хранить свои переводы

Простой бэкенд, поставляющийся вместе с Active Support, позволяет хранить переводы как в формате чистого Ruby, так и в YAML. (Другие бэкенды могут позволить или требовать использование иных форматов, например GetText позволяет использовать формат GetText.)

Например, представляющий перевод хэш Ruby выглядит так:

```ruby
{
  :pt => {
    :foo => {
      :bar => "baz"
    }
  }
}
```

Эквивалентный файл YAML выглядит так:

```yaml
pt:
  foo:
    bar: baz
```

Как видите, в обоих случаях ключ верхнего уровня является локалью. `:foo` - это ключ пространства имен, а `:bar` - это ключ для перевода "baz".

Вот "реальный" пример из YAML файла перевода Active Support `en.yml`:

```yaml
en:
  date:
    formats:
      default: "%Y-%m-%d"
      short: "%b %d"
      long: "%B %d, %Y"
```

Таким образом, все из нижеследующих эквивалентов возвратит короткий (`:short`) формат даты `"%B %d"`:

```ruby
I18n.t 'date.formats.short'
I18n.t 'formats.short', :scope => :date
I18n.t :short, :scope => 'date.formats'
I18n.t :short, :scope => [:date, :formats]
```

Как правило мы рекомендуем использовать YAML как формат хранения переводов. Хотя имеются случаи, когда хочется хранить лямбда-функции Ruby как часть данных локали, например, для специальных форматов дат.

### Переводы для моделей Active Record

Можете использовать методы `Model.human_name` и `Model.human_attribute_name(attribute)` для прозрачного поиска переводов для ваших моделей и имен атрибутов.

Например, когда добавляем следующие переводы:

```yaml
en:
  activerecord:
    models:
      user: Dude
    attributes:
      user:
        login: "Handle"
      # will translate User attribute "login" as "Handle"
```

Тогда `User.human_name` возвратит "Dude", а `User.human_attribute_name("login")` возвратит "Handle".

#### Пространства имен сообщений об ошибке

Сообщение об ошибке валидации Active Record также может быть легко переведено. Active Record предоставляет ряд пространств имен, куда можно поместить ваши переводы для передачи различных сообщений и переводы для определенных моделей, аттрибутов и/или валидаций. Также учитывается одиночное наследование таблицы (single table inheritance).

Это дает довольно мощное средство для гибкой настройки ваших сообщений в соответствии с потребностями приложения.

Рассмотрим модель User с валидацией `validates_presence_of` для атрибута name, подобную следующей:

```ruby
class User < ActiveRecord::Base
  validates :name, :presence => true
end
```

Ключом для сообщения об ошибке в этом случае будет `:blank`. Active Record будет искать этот ключ в пространствах имен:

```ruby
activerecord.errors.models.[model_name].attributes.[attribute_name]
activerecord.errors.models.[model_name]
activerecord.errors.messages
errors.attributes.[attribute_name]
errors.messages
```

Таким образом, в нашем примере он будет перебирать следующие ключи в указанном порядке и возвратит первый полученный результат:

```ruby
activerecord.errors.models.user.attributes.name.blank
activerecord.errors.models.user.blank
activerecord.errors.messages.blank
errors.attributes.name.blank
errors.messages.blank
```

Когда ваши модели дополнительно используют наследование, тогда сообщения ищутся в цепочке наследования.

Например, у вас может быть модель Admin, унаследованная от User:

```ruby
class Admin < User
  validates :name, :presence => true
end
```

Тогда Active Record будет искать сообщения в этом порядке:

```ruby
activerecord.errors.models.admin.attributes.name.blank
activerecord.errors.models.admin.blank
activerecord.errors.models.user.attributes.name.blank
activerecord.errors.models.user.blank
activerecord.errors.messages.blank
errors.attributes.name.blank
errors.messages.blank
```

Таким образом можно предоставить специальные переводы для различных сообщений об ошибке в различных местах цепочки наследования моделей и в атрибутах, моделях и пространствах имен по умолчанию.

#### Интерполяция сообщения об ошибке

Переведенное имя модели, переведенное имя атрибута и значение всегда доступны для интерполяции.

Так, к примеру, вместо сообщения об ошибке по умолчанию `"can not be blank"` можете использовать имя атрибута как тут: `"Please fill in your %{attribute}"`.

* Где это возможно, `count` может быть использован для множественного числа, если оно существует:

| валидация    | с опцией                  | сообщение                 | интерполяция |
| ------------ | ------------------------- | ------------------------- | ------------ |
| confirmation | -                         | :confirmation             | -            |
| acceptance   | -                         | :accepted                 | -            |
| presence     | -                         | :blank                    | -            |
| length       | :within, :in              | :too_short                | count        |
| length       | :within, :in              | :too_long                 | count        |
| length       | :is                       | :wrong_length             | count        |
| length       | :minimum                  | :too_short                | count        |
| length       | :maximum                  | :too_long                 | count        |
| uniqueness   | -                         | :taken                    | -            |
| format       | -                         | :invalid                  | -            |
| inclusion    | -                         | :inclusion                | -            |
| exclusion    | -                         | :exclusion                | -            |
| associated   | -                         | :invalid                  | -            |
| numericality | -                         | :not_a_number             | -            |
| numericality | :greater_than             | :greater_than             | count        |
| numericality | :greater_than_or_equal_to | :greater_than_or_equal_to | count        |
| numericality | :equal_to                 | :equal_to                 | count        |
| numericality | :less_than                | :less_than                | count        |
| numericality | :less_than_or_equal_to    | :less_than_or_equal_to    | count        |
| numericality | :odd                      | :odd                      | -            |
| numericality | :even                     | :even                     | -            |

#### Переводы для хелпера Active Record `error_messages_for`

Если используете хелпер Active Record `error_messages_for`, то, возможно, захотите добавить для него переводы.

Rails поставляется со следующими переводами:

```yaml
en:
  activerecord:
    errors:
      template:
        header:
          one:   "1 error prohibited this %{model} from being saved"
          other: "%{count} errors prohibited this %{model} from being saved"
        body:    "There were problems with the following fields:"
```

### Обзор других встроенных методов, предоставляющих поддержку I18n

Rails использует фиксированные строки и другие локализации, такие как формат строки и другая информация о формате, в ряде хелперов. Вот краткий обзор.

#### Методы хелпера Action View

* `distance_of_time_in_words` переводит и образует множественное число своего результата и интерполирует число секунд, минут, часов и т.д. Смотрите переводы [datetime.distance_in_words](http://github.com/rails/rails/blob/master/actionpack/lib/action_view/locale/en.yml#L51).
* `datetime_select` и `select_month` используют переведенные имена месяцев для заполнения результирующего тега select. Смотрите переводы в [date.month_names](http://github.com/rails/rails/blob/master/activesupport/lib/active_support/locale/en.yml#L15). `datetime_select` также ищет опцию order из [date.order](http://github.com/rails/rails/blob/master/activesupport/lib/active_support/locale/en.yml#L18) (если вы передали эту опцию явно). Все хелперы выбора даты переводят prompt, используя переводы в пространстве имен [datetime.prompts](http://github.com/rails/rails/blob/master/actionpack/lib/action_view/locale/en.yml#L83), если применимы.
* Хелперы `number_to_currency`, `number_with_precision`, `number_to_percentage`, `number_with_delimiter` и `number_to_human_size` используют настройки формата чисел в пространстве имен [number](http://github.com/rails/rails/blob/master/actionpack/lib/action_view/locale/en.yml#L2).

#### Методы Active Model

* `human_name` и `human_attribute_name` используют переводы для имен модели и имен аттрибутов, если они доступны в пространстве имен [activerecord.models](http://github.com/rails/rails/blob/master/activerecord/lib/active_record/locale/en.yml#L29). Они также предоставляют переводы для имен унаследованного класса (т.е. для использования вместе с STI), как уже объяснялось выше в "Области сообщения об ошибке".
* `ActiveModel::Errors#generate_message` (который используется валидациями Active Model, но также может быть использован вручную) использует `human_name` и `human_attribute_name` (смотрите выше). Он также переводит сообщение об ошибке и поддерживает переводы для имен унаследованного класса, как уже объяснялось выше в "Пространства имен сообщений об ошибке".
* `ActiveModel::Errors#full_messages` добавляет имя атрибута к сообщению об ошибке, используя разделитель, который берется из [errors.format](https://github.com/rails/rails/blob/master/activemodel/lib/active_model/locale/en.yml#L4) (и по умолчанию равен `"%{attribute} %{message}"`).

#### Методы Active Support

* `Array#to_sentence` использует настройки формата, которые заданы в пространстве имен [support.array](http://github.com/rails/rails/blob/master/activesupport/lib/active_support/locale/en.yml#L30).
