Основы Active Job
=================

Это руководство даст вам все, что нужно, чтобы начать создавать, ставить в очередь и запускать фоновые задачи.

После его прочтения, вы узнаете:

* Как создавать задачи.
* Как ставить в очередь задачи.
* Как запускать задачи в фоне.
* Как асинхронно рассылать письма из вашего приложения.

--------------------------------------------------------------------------------


Введение
------------

Active Job - это фреймворк для объявления задач и их запуска на разных бэкендах для очередей. Эти задачи могут быть чем угодно, от регулярно запланированных чисток до списаний с карт или рассылок. В общем, всем, что может быть выделено в небольшие работающие части и запускаться параллельно.


Назначение Active Job
---------------------

Главным является то, что он обеспечивает, что у всех приложений Rails имеется встроенная инфраструктура для задач. Затем у нас могут появиться особенности фреймворка или других гемов, созданных на его основе, позволяющие не заботится об отличиях в API между различными исполнителями задач, такими как Delayed Job и Resque. Подбор бэкенда для очередей станет более оперативной работой. Вы сможете переключаться между ними без необходимости переписывать свои задачи.

NOTE: По умолчанию, Rails поставляется с реализацией очереди в виде "немедленного исполнения".
Это означает, что каждая задача, поставленная в очередь, будет запущена сразу.

Создание задачи
---------------

Этот раздел представляет пошаговое руководство для создании задачи и добавления ее в очередь.

### Создание задачи

Active Job предоставляет генератор Rails для создания задач. Следующая команда создаст задачу в `app/jobs` (а также тестовый случай в `test/jobs`):

```bash
$ bin/rails generate job guests_cleanup
invoke  test_unit
create    test/jobs/guests_cleanup_job_test.rb
create  app/jobs/guests_cleanup_job.rb
```

Также можно создать задачу, которая будет запущена в определенной очереди:

```bash
$ bin/rails generate job guests_cleanup --queue urgent
```

Если не хотите использовать генератор, можно создать файл очереди в `app/jobs`, просто убедитесь, что он наследуется от `ActiveJob::Base`.

Вот как выглядит задача:

```ruby
class GuestsCleanupJob < ActiveJob::Base
  queue_as :default

  def perform(*args)
    # Do something later
  end
end
```

### Помещение задачи в очередь

Поместить задачу в очередь можно так:

```ruby
# Помещенная в очередь задача выполнится, как только освободится система очередей.
MyJob.perform_later record
```

```ruby
# Помещенная в очередь задача выполнится завтра в полдень.
MyJob.set(wait_until: Date.tomorrow.noon).perform_later(record)
```

```ruby
# Помещенная в очередь задача выполнится через неделю.
MyJob.set(wait: 1.week).perform_later(record)
```

Вот и все!

Запуск задач
------------

Чтобы поместить задачу в очередь и выполнить ее, вам необходимо настроить бэкенд для очереди, т.е. вам нужно решить, какую стороннюю библиотеку для очереди Rails будет использовать. Rails не предоставляет сложную систему для работы с очередями, а просто выполняет задачу немедленно, если не настроен какой-либо адаптер.

### Бэкенды

У Active Job есть встроенные адаптеры для различных бэкендов очередей (Sidekiq, Resque, Delayed Job и другие). Чтобы получить актуальный список адаптеров, обратитесь к документации API по [ActiveJob::QueueAdapters](http://api.rubyonrails.org/classes/ActiveJob/QueueAdapters.html).

### Настройка бэкенда

Настроить бэкенд — это просто:

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    # Убедитесь, что гем адаптера добавлен в Gemfile, и что выполнены
    # инструкции по установке и развертыванию адаптера.
    config.active_job.queue_adapter = :sidekiq
  end
end
```

NOTE: Поскольку задачи запускаются параллельно с вашим Rails приложением, большинство библиотек для работы с очередями требуют запуска специфичной для библиотеки службы очереди (помимо старта вашего Rails приложения) для обработки задач. Для получения информации о том, как это сделать, обратитесь к документации соответствующей библиотеки.

Очереди
-------

Большая часть адаптеров поддерживает несколько очередей. С помощью Active Job можно запланировать, что задача будет выполнена в определенной очереди:

```ruby
class GuestsCleanupJob < ActiveJob::Base
  queue_as :low_priority
  #....
end
```

Можно задать префикс для имени очереди для всех задач с помощью `config.active_job.queue_name_prefix` в `application.rb`:

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    config.active_job.queue_name_prefix = Rails.env
  end
end

# app/jobs/guests_cleanup.rb
class GuestsCleanupJob < ActiveJob::Base
  queue_as :low_priority
  #....
end

# Теперь ваша задача запустится в очереди production_low_priority в среде
# production и в staging_low_priority в среде staging
```

Разделитель префикса имени очереди по умолчанию '\_'.  Его можно изменить, установив `config.active_job.queue_name_delimiter` в `application.rb`:

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    config.active_job.queue_name_prefix = Rails.env
    config.active_job.queue_name_delimiter = '.'
  end
end

# app/jobs/guests_cleanup.rb
class GuestsCleanupJob < ActiveJob::Base
  queue_as :low_priority
  #....
end

# Теперь ваша задача запустится в очереди production.low_priority в среде
# production и в staging.low_priority в среде staging
```

Если хотите больше контроля, в какой очереди задача будет запущена, можно передать опцию `:queue` в `#set`:

```ruby
MyJob.set(queue: :another_queue).perform_later(record)
```

Чтобы контролировать очередь на уровне задачи, можно передать блок в `#queue_as`. Блок будет выполнен в контексте задачи (таким образом, у вас будет доступ к `self.arguments`), и он должен вернуть имя очереди:

```ruby
class ProcessVideoJob < ActiveJob::Base
  queue_as do
    video = self.arguments.first
    if video.owner.premium?
      :premium_videojobs
    else
      :videojobs
    end
  end

  def perform(video)
    # Делаем обработку видео
  end
end

ProcessVideoJob.perform_later(Video.last)
```

NOTE: Убедитесь, что ваш бэкенд для очередей "слушает" имя вашей очереди. Для некоторых бэкендов необходимо указать очереди, которые нужно слушать.

Колбэки
-------

Active Job предоставляет хуки на протяжение жизненного цикла задачи. Колбэки позволяют включить логику в жизненный цикл задачи.

### Доступные колбэки

* `before_enqueue`
* `around_enqueue`
* `after_enqueue`
* `before_perform`
* `around_perform`
* `after_perform`

### Использование

```ruby
class GuestsCleanupJob < ActiveJob::Base
  queue_as :default

  before_enqueue do |job|
    # делаем что-то с экземпляром задачи
  end

  around_perform do |job, block|
    # делаем что-то перед выполнением
    block.call
    # делаем что-то после выполнения
  end

  def perform
    # Отложенная задача
  end
end
```

Action Mailer
------------

Одной из обычных задач в современном веб-приложении является рассылка писем за пределами цикла запроса-отклика, чтобы пользователь не ждал. Active Job интегрируется с Action Mailer, поэтому рассылать письма асинхронно очень просто:

```ruby
# Если хотите отправить письмо сейчас, используйте #deliver_now
UserMailer.welcome(@user).deliver_now

# Если хотите отправить письмо через Active Job, используйте #deliver_later
UserMailer.welcome(@user).deliver_later
```

Интернационализация
-------------------

Каждая задача использует `I18n.locale` при создание. Это полезно, если вы отправляете письма асинхронно:

```ruby
I18n.locale = :eo
 
UserMailer.welcome(@user).deliver_later # Email будет локализован в Эсперанто.
```

GlobalID
--------

Active Job поддерживает GlobalID для параметров. Это позволяет передавать объекты Active Record в ваши задачи, вместо пар класс/id, которые нужно затем десериализовать вручную. Раньше задачи выглядели так:

```ruby
class TrashableCleanupJob < ActiveJob::Base
  def perform(trashable_class, trashable_id, depth)
    trashable = trashable_class.constantize.find(trashable_id)
    trashable.cleanup(depth)
  end
end
```

Теперь можно просто сделать так:

```ruby
class TrashableCleanupJob < ActiveJob::Base
  def perform(trashable, depth)
    trashable.cleanup(depth)
  end
end
```

Это работает с любым классом, в который подмешан `GlobalID::Identification`, который по умолчанию был подмешан в классы Active Record.

Исключения
----------

Active Job предоставляет способ отлова исключений, возникших в течение запуска задачи:

```ruby

class GuestsCleanupJob < ActiveJob::Base
  queue_as :default

  rescue_from(ActiveRecord::RecordNotFound) do |exception|
   # Сделать что-то с этим исключением
  end

  def perform
    # Отложенная задача
  end
end
```

### Десериализация

GlobalID позволяет сериализовать полностью объекты Active Record, переданные в `#perform`.

Если переданная запись была удалена после того, как задача была помещена в очередь, но до того, как метод `#perform` был вызван, Active Job вызовет исключение `ActiveJob::DeserializationError`.

Тестирование задач
------------------

Вы можете найти подробные инструкции о том, как тестировать ваши задачи в [руководстве по тестированию](testing.html#testing-jobs).
