# Расширения для Module

### `alias_method_chain`

Используя чистый Ruby можно обернуть методы в другие методы, это называется _сцепление псевдонимов (alias chaining)_.

Например, скажем, что вы хотите, чтобы params были строками в функциональных тестах, как и в реальных запросах, но также хотите удобно присваивать числа и другие типы значений. Чтобы это осуществить, следует обернуть `ActionController::TestCase#process` следующим образом в `test/test_helper.rb`:

```ruby
ActionController::TestCase.class_eval do
  # сохраняем ссылку на оригинальный метод process
  alias_method :original_process, :process

  # теперь переопределяем process и передаем в original_process
  def process(action, params=nil, session=nil, flash=nil, http_method='GET')
    params = Hash[*params.map {|k, v| [k, v.to_s]}.flatten]
    original_process(action, params, session, flash, http_method)
  end
end
```

Таким образом передают работу методы `get`, `post` и т.д..

В такой технике имеется риск, в случае если `:original_process` уже есть. Чтобы избежать коллизий, некоторые выбирают определенные метки, характеризующие то, что сцепление означает:

```ruby
ActionController::TestCase.class_eval do
  def process_with_stringified_params(...)
    params = Hash[*params.map {|k, v| [k, v.to_s]}.flatten]
    process_without_stringified_params(action, params, session, flash, http_method)
  end
  alias_method :process_without_stringified_params, :process
  alias_method :process, :process_with_stringified_params
end
```

Метод `alias_method_chain` предоставляет ярлык для такого примера:

```ruby
ActionController::TestCase.class_eval do
  def process_with_stringified_params(...)
    params = Hash[*params.map {|k, v| [k, v.to_s]}.flatten]
    process_without_stringified_params(action, params, session, flash, http_method)
  end
  alias_method_chain :process, :stringified_params
end
```

Rails использует `alias_method_chain` во всем своем коде. Например, валидации добавляются в `ActiveRecord::Base#save` через оборачивания метода подобным образом в отдельный модуль, специализирующийся на валидациях.

NOTE: Определено в `active_support/core_ext/module/aliasing.rb`.

### Атрибуты

#### `alias_attribute`

В атрибутах модели есть ридер (reader), райтер (writer), и условие (predicate). Можно создать псевдоним к атрибуту модели, имеющему соответствующие три метода, за раз. Как и в других создающих псевдоним методах, новое имя - это первый аргумент, а старое имя - второй (мое мнемоническое правило такое: они идут в том же порядке, как если бы делалось присваивание):

```ruby
class User < ActiveRecord::Base
  # давайте назовем колонку email как "login",
  # что более значимо для аутентификационного кода
  alias_attribute :login, :email
end
```

NOTE: Определено в `active_support/core_ext/module/aliasing.rb`.

#### Внутренние атрибуты

При определении атрибута в классе есть риск коллизий субклассовых имен. Это особенно важно для библиотек.

Active Support определяет макросы `attr_internal_reader`, `attr_internal_writer` и `attr_internal_accessor`. Они ведут себя подобно встроенным в Ruby коллегам `attr_*`, за исключением того, что они именуют лежащую в основе переменную экземпляра способом, наиболее снижающим коллизии.

Макрос `attr_internal` - это синоним для `attr_internal_accessor`:

```ruby
# библиотека
class ThirdPartyLibrary::Crawler
  attr_internal :log_level
end

# код клиента
class MyCrawler < ThirdPartyLibrary::Crawler
  attr_accessor :log_level
end
```

В предыдущем примере мог быть случай, что `:log_level` не принадлежит публичному интерфейсу библиотеки и используется только для разработки. Код клиента, не знающий о потенциальных конфликтах, субклассифицирует и определяет свой собственный `:log_level`. Благодаря `attr_internal` здесь нет коллизий.

По умолчанию внутренняя переменная экземпляра именуется с предшествующим подчеркиванием, `@_log_level` в примере выше. Это настраивается через `Module.attr_internal_naming_format`, куда можно передать любую строку в формате `sprintf` с предшествующим `@` и `%s` в любом месте, которая означает место, куда вставляется имя. По умолчанию `"@_%s"`.

Rails использует внутренние атрибуты в некоторых местах, например для вьюх:

```ruby
module ActionView
  class Base
    attr_internal :captures
    attr_internal :request, :layout
    attr_internal :controller, :template
  end
end
```

NOTE: Определено в `active_support/core_ext/module/attr_internal.rb`.

#### Атрибуты модуля

Макросы `mattr_reader`, `mattr_writer` и `mattr_accessor` - это аналоги макросам `cattr_*`, определенным для класса. Смотрите [Атрибуты класса](/active-support-core-extensions/extensions-to-class).

Например, их использует механизм зависимостей:

```ruby
module ActiveSupport
  module Dependencies
    mattr_accessor :warnings_on_first_load
    mattr_accessor :history
    mattr_accessor :loaded
    mattr_accessor :mechanism
    mattr_accessor :load_paths
    mattr_accessor :load_once_paths
    mattr_accessor :autoloaded_constants
    mattr_accessor :explicitly_unloadable_constants
    mattr_accessor :logger
    mattr_accessor :log_activity
    mattr_accessor :constant_watch_stack
    mattr_accessor :constant_watch_stack_mutex
  end
end
```

NOTE: Определено в `active_support/core_ext/module/attribute_accessors.rb`.

### Родители

#### `parent`

Метод `parent` на вложенном именованном модуле возвращает модуль, содержащий его соответствующую константу:

```ruby
module X
  module Y
    module Z
    end
  end
end
M = X::Y::Z

X::Y::Z.parent # => X::Y
M.parent       # => X::Y
```

Если модуль анонимный или относится к верхнему уровню, `parent` возвращает `Object`.

WARNING: Отметьте, что в этом случае `parent_name` возвращает `nil`.

NOTE: Определено в `active_support/core_ext/module/introspection.rb`.

#### `parent_name`

Метод `parent_name` на вложенном именованном модуле возвращает полное имя модуля, содержащего его соответствующую константу:

```ruby
module X
  module Y
    module Z
    end
  end
end
M = X::Y::Z

X::Y::Z.parent_name # => "X::Y"
M.parent_name       # => "X::Y"
```

Для модулей верхнего уровня и анонимных `parent_name` возвращает `nil`.

WARNING: Отметьте, что в этом случае `parent` возвращает `Object`.

NOTE: Определено в `active_support/core_ext/module/introspection.rb`.

#### `parents`

Метод `parents` вызывает `parent` на получателе и выше, пока не достигнет `Object`. Цепочка возвращается в массиве, от низшего к высшему:

```ruby
module X
  module Y
    module Z
    end
  end
end
M = X::Y::Z

X::Y::Z.parents # => [X::Y, X, Object]
M.parents       # => [X::Y, X, Object]
```

NOTE: Определено в `active_support/core_ext/module/introspection.rb`.

### Константы

Метод `local_constants` возвращает имена констант, которые были определены в модуле получателя:

```ruby
module X
  X1 = 1
  X2 = 2
  module Y
    Y1 = :y1
    X1 = :overrides_X1_above
  end
end

X.local_constants    # => [:X1, :X2, :Y]
X::Y.local_constants # => [:Y1, :X1]
```

Имена возвращаются как символы. (Устаревший метод `local_constant_names` возвращает строки.)

NOTE: Определено в `active_support/core_ext/module/introspection.rb`.

#### Qualified Constant Names

Стандартные методы `const_defined?`, `const_get` и `const_set` принимают простые имена констант. Active Support расширяет это API на передачу полных имен констант.

Новые методы - это `qualified_const_defined?`, `qualified_const_get` и `qualified_const_set`. Их аргументами предполагаются полные имена констант относительно получателя:

```ruby
Object.qualified_const_defined?("Math::PI")       # => true
Object.qualified_const_get("Math::PI")            # => 3.141592653589793
Object.qualified_const_set("Math::Phi", 1.618034) # => 1.618034
```

Аргументы могут быть и простыми именами констант:

```ruby
Math.qualified_const_get("E") # => 2.718281828459045
```

Эти методы аналогичны их встроенным коллегам. В частности, `qualified_constant_defined?` принимает опциональный второй аргумент, указывающий, хотите ли вы, чтобы этот метод искал в предках. Этот флажок учитывается для каждой константы в выражении во время прохода.

Для примера, дано

```ruby
module M
  X = 1
end

module N
  class C
    include M
  end
end
```

`qualified_const_defined?` ведет себя таким образом:

```ruby
N.qualified_const_defined?("C::X", false) # => false
N.qualified_const_defined?("C::X", true)  # => true
N.qualified_const_defined?("C::X")        # => true
```

Как показывает последний пример, второй аргумент по умолчанию true, как и в `const_defined?`.

Для согласованности со встроенными методами принимаются только относительные пути. Абсолютные полные имена констант, такие как `::Math::PI`, вызывают `NameError`.

NOTE: Определено в `active_support/core_ext/module/qualified_const.rb`.

### Reachable

Именнованный модуль является достижимым (reachable), если он хранится в своей соответствующей константе. Это означает, что можно связаться с объектом модуля через константу.

Это означает, что если есть модуль с названием "M", то существует константа `M`, которая указывает на него:

```ruby
module M
end

M.reachable? # => true
```

Но так как константы и модули в действительности являются разъединенными, объекты модуля могут стать недостижимыми:

```ruby
module M
end

orphan = Object.send(:remove_const, :M)

# Теперь объект модуля это orphan, но у него все еще есть имя.
orphan.name # => "M"

# Нельзя достичь его через константу M, поскольку она даже не существует.
orphan.reachable? # => false

# Давайте определим модуль с именем "M" снова.
module M
end

# Теперь константа M снова существует, и хранит объект
# модуля с именем "M", но это новый экземпляр.
orphan.reachable? # => false
```

NOTE: Определено в `active_support/core_ext/module/reachable.rb`.

### Anonymous

Модуль может иметь или не иметь имени:

```ruby
module M
end
M.name # => "M"

N = Module.new
N.name # => "N"

Module.new.name # => nil
```

Можно проверить, имеет ли модуль имя с помощью условия `anonymous?`:

```ruby
module M
end
M.anonymous? # => false

Module.new.anonymous? # => true
```

Отметьте, что быть недоступным не означает быть анонимным:

```ruby
module M
end

m = Object.send(:remove_const, :M)

m.reachable? # => false
m.anonymous? # => false
```

хотя анонимный модуль недоступен по определению.

NOTE: Определено в `active_support/core_ext/module/anonymous.rb`.

### Передача метода

Макрос `delegate` предлагает простой способ передать методы.

Давайте представим, что у пользователей в неком приложении имеется информация о логинах в модели `User`, но имена и другие данные в отдельной модели `Profile`:

```ruby
class User < ActiveRecord::Base
  has_one :profile
end
```

С такой конфигурацией имя пользователя получается через его профиль, `user.profile.name`, но можно обеспечить прямой доступ как к атрибуту:

```ruby
class User < ActiveRecord::Base
  has_one :profile

  def name
    profile.name
  end
end
```

Это как раз то, что делает `delegate`:

```ruby
class User < ActiveRecord::Base
  has_one :profile

  delegate :name, :to => :profile
end
```

Это короче, и намерения более очевидные.

Целевой метод должен быть публичным.

Макрос `delegate` принимает несколько методов:

```ruby
delegate :name, :age, :address, :twitter, :to => :profile
```

При интерполяции в строку опция `:to` должна стать выражением, применяемым к объекту, метод которого передается. Обычно строка или символ. Такое выражение вычисляется в контексте получателя:

```ruby
# передает константе Rails
delegate :logger, :to => :Rails

# передает классу получателя
delegate :table_name, :to => 'self.class'
```

WARNING: Если опция `:prefix` установлена `true` это менее характерно, смотрите ниже.

По умолчанию, если передача вызывает `NoMethodError` и цель является `nil`, выводится исключение. Можно попросить, чтобы возвращался `nil` с помощью опции `:allow_nil`:

```ruby
delegate :name, :to => :profile, :allow_nil => true
```

С `:allow_nil` вызов `user.name` возвратит `nil`, если у пользователя нет профиля.

Опция `:prefix` добавляет префикс к имени генерируемого метода. Это удобно, если хотите получить более благозвучное наименование:

```ruby
delegate :street, :to => :address, :prefix => true
```

Предыдущий пример создаст `address_street`, а не `street`.

WARNING: Поскольку в этом случае имя создаваемого метода составляется из имен целевого объекта и целевого метода, опция `:to` должна быть именем метода.

Также может быть настроен произвольный префикс:

```ruby
delegate :size, :to => :attachment, :prefix => :avatar
```

В предыдущем примере макрос создаст `avatar_size`, а не `size`.

NOTE: Определено в `active_support/core_ext/module/delegation.rb`

### Переопределение методов

Бывают ситуации, когда нужно определить метод с помощью `define_method`, но вы не знаете, существует ли уже метод с таким именем. Если так, то выдается предупреждение, если оно включено. Такое поведение хоть и не ошибочно, но не элегантно.

Метод `redefine_method` предотвращает такое потенциальное предупреждение, предварительно убирая существующий метод, если нужно. Rails использует это в некоторых местах, к примеру когда он создает `API` по связям:

```ruby
redefine_method("#{reflection.name}=") do |new_value|
  association = association_instance_get(reflection.name)

  if association.nil? || association.target != new_value
    association = association_proxy_class.new(self, reflection)
  end

  association.replace(new_value)
  association_instance_set(reflection.name, new_value.nil? ? nil : association)
end
```

NOTE: Определено в `active_support/core_ext/module/remove_method.rb`
