# Обзор особенностей I18n API

Теперь у вас есть хорошее понимание об использовании библиотеки i18n, знания всех необходимых аспектов интернационализации простого приложения на Rails. В следующих частях мы раскроем особенности более детально.

Раскроем особенности такие, как:

* поиск переводов
* интерполяция данных в переводы
* множественное число у переводов
* использование HTML-безопасных переводов
* локализация дат, номеров, валют и т.п.

### Поиск переводов

#### Основы поиска, области имен и вложенных ключей

Переводы ищутся по ключам, которые могут быть как символами, так и строками, поэтому следующие вызовы эквивалентны:

```ruby
I18n.t :message
I18n.t 'message'
```

Метод `translate` также принимает опцию `:scope`, которая содержит один или более дополнительных ключей, которые будут использованы для определения “пространства” или области имен для ключа перевода:

```ruby
I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]
```

Тут будет искаться сообщение `:record_invalid` в сообщениях об ошибке Active Record.

Кроме того, и ключ, и область имен могут быть определены как ключи с точкой в качестве разделителя, как в:

```ruby
I18n.translate "activerecord.errors.messages.record_invalid"
```

Таким образом, следующие вызовы эквивалентны:

```ruby
I18n.t 'activerecord.errors.messages.record_invalid'
I18n.t 'errors.messages.record_invalid', :scope => :active_record
I18n.t :record_invalid, :scope => 'activerecord.errors.messages'
I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]
```

#### Значения по умолчанию

Когда задана опция `:default`, будет возвращено ее значение в случае, если отсутствует перевод:

```ruby
I18n.t :missing, :default => 'Not here'
# => 'Not here'
```

Если значение `:default` является символом, оно будет использовано как ключ и будет переведено. Может быть представлено несколько значений по умолчанию. Будет возвращено первое, которое даст результат.

Т.е., следующее попытается перевести ключ `:missing`, затем ключ `:also_missing`. Если они оба не дадут результат, будет возвращена строка "Not here":

```ruby
I18n.t :missing, :default => [:also_missing, 'Not here']
# => 'Not here'
```

#### Массовый поиск и поиск в пространстве имен

Чтобы найти несколько переводов за раз, может быть передан массив ключей:

```ruby
I18n.t [:odd, :even], :scope => 'errors.messages'
# => ["must be odd", "must be even"]
```

Также, ключ может перевести хэш (потенциально вложенный) сгруппированных переводов. Т.е. следующее получит _все_ сообщения об ошибке Active Record как хэш:

```ruby
I18n.t 'activerecord.errors.messages'
# => { :inclusion => "is not included in the list", :exclusion => ... }
```

#### "Ленивый" поиск

Rails реализует удобный способ поиска локали внутри _вьюх_. Когда имеется следующий словарь:

```yaml
es:
  books:
    index:
      title: "Título"
```

можно найти значение `books.index.title` **в** шаблоне `app/views/books/index.html.erb` таким образом (обратите внимание на точку):

```ruby
<%= t '.title' %>
```

### Интерполяция

Во многих случаях хочется абстрагировать свои переводы так, чтобы **переменные могли быть интерполированы в переводы**. В связи с этим, API I18n предоставляет особенность интерполяции.

Все опции, кроме `:default` и `:scope`, которые передаются в `#translate`, будут интерполированы в перевод:

```ruby
I18n.backend.store_translations :en, :thanks => 'Thanks %{name}!'
I18n.translate :thanks, :name => 'Jeremy'
# => 'Thanks Jeremy!'
```

Если перевод использует `:default` или `:scope` как интерполяционную переменную, будет вызвано исключение `I18n::ReservedInterpolationKey`. Если перевод ожидает интерполяционную переменную, но она не была передана в `#translate`, вызовется исключение `I18n::MissingInterpolationArgument`.

### Множественное число

В английском только одна форма единственного числа, и одна множественного для заданной строки, т.е. "1 message" и "2 messages". В других языках ([русском](http://unicode.org/repos/cldr-tmp/trunk/diff/supplemental/language_plural_rules.html#ru), [арабском](http://unicode.org/repos/cldr-tmp/trunk/diff/supplemental/language_plural_rules.html#ar), [японском](http://unicode.org/repos/cldr-tmp/trunk/diff/supplemental/language_plural_rules.html#ja) и многих других) имеются различные правила грамматики, имеющие дополнительные или остутствующие [формы множественного числа](http://unicode.org/repos/cldr-tmp/trunk/diff/supplemental/language_plural_rules.html). Таким образом, API I18n предоставляет гибкую возможность множественных форм.

У переменной интерполяции `:count` есть специальная роль в том, что она интерполируется для перевода, и используется для подбора множественного числа для перевода в соответствии с правилами множественного числа, определенными в CLDR:

```ruby
I18n.backend.store_translations :en, :inbox => {
  :one => 'one message',
  :other => '%{count} messages'
}
I18n.translate :inbox, :count => 2
# => '2 messages'

I18n.translate :inbox, :count => 1
# => 'one message'
```

Алгоритм для образования множественного числа в `:en` прост:

```ruby
entry[count == 1 ? 0 : 1]
```

Т.е., перевод помеченный как `:one`, рассматривается как единственное число, все другое как множественное (включая ноль).

Если поиск по ключу не возвратит хэш, подходящий для образования множественного числа, вызовется исключение `18n::InvalidPluralizationData`.

### Настройка и передача локали

Локаль может быть либо установленной псевдо-глобально в `I18n.locale` (когда используется `Thread.current`, например `Time.zone`), либо быть переданной опцией в `#translate` и `#localize`.

Если локаль не была передана, используется `I18n.locale`:

```ruby
I18n.locale = :de
I18n.t :foo
I18n.l Time.now
```

Явно переданная локаль:

```ruby
I18n.t :foo, :locale => :de
I18n.l Time.now, :locale => :de
```

Умолчанием для `I18n.locale` является `I18n.default_locale`, для которой по умолчанию установлено `:en`. Локаль по умолчанию может быть установлена так:

```ruby
I18n.default_locale = :de
```

### Использование HTML-безопасных переводов

Ключи с суффиксом '_html' и ключами с именем 'html' помечаются как HTML-безопасные. Их можно использовать во вьюхах без экранирования.

```yaml
# config/locales/en.yml
en:
  welcome: <b>welcome!</b>
  hello_html: <b>hello!</b>
  title:
    html: <b>title!</b>
```

```erb
# app/views/home/index.html.erb
<div><%= t('welcome') %></div>
<div><%= raw t('welcome') %></div>
<div><%= t('hello_html') %></div>
<div><%= t('title.html') %></div>
```

![демонстрация html-безопасности в i18n](/assets/guides/i18n_demo_html_safe.png)
