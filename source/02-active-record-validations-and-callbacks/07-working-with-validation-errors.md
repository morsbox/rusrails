# Работаем с ошибками валидации

В дополнение к методам `valid?` и `invalid?`, раскрытым ранее, Rails предоставляет ряд методов для работы с коллекцией `errors` и исследования валидности объектов.

Предлагаем список наиболее часто используемых методов. Если хотите увидеть список всех доступных методов, обратитесь к документации по `ActiveModel::Errors`.

### `errors`

Возвращает экземпляр класса `ActiveModel::Errors`, содержащий все ошибки. Каждый ключ это имя атрибута и значение это массив строк со всеми ошибками.

```ruby
class Person < ActiveRecord::Base
  validates :name, :presence => true, :length => { :minimum => 3 }
end

person = Person.new
person.valid? # => false
person.errors
 # => {:name => ["can't be blank", "is too short (minimum is 3 characters)"]}

person = Person.new(:name => "John Doe")
person.valid? # => true
person.errors # => []
```

### `errors[]`

`errors[]` используется, когда вы хотите проверить сообщения об ошибке для определенного атрибута. Он возвращает массив строк со всеми сообщениями об ошибке для заданного атрибута, каждая строка с одним сообщением об ошибке. Если нет ошибок, относящихся к атрибуту, возвратится пустой массив.

```ruby
class Person < ActiveRecord::Base
  validates :name, :presence => true, :length => { :minimum => 3 }
end

person = Person.new(:name => "John Doe")
person.valid? # => true
person.errors[:name] # => []

person = Person.new(:name => "JD")
person.valid? # => false
person.errors[:name] # => ["is too short (minimum is 3 characters)"]

person = Person.new
person.valid? # => false
person.errors[:name]
 # => ["can't be blank", "is too short (minimum is 3 characters)"]
```

### `errors.add`

Метод `add` позволяет вручную добавлять сообщения, которые относятся к определенным атрибутам. Можно использовать методы `errors.full_messages` или `errors.to_a` для просмотра сообщения в форме, в которой они отображаются пользователю. Эти определенные сообщения получают предшествующим (и с прописной буквы) имя аттрибута. `add` получает имя атрибута, к которому вы хотите добавить сообщение, и само сообщение.

```ruby
class Person < ActiveRecord::Base
  def a_method_used_for_validation_purposes
    errors.add(:name, "cannot contain the characters !@#%*()_-+=")
  end
end

person = Person.create(:name => "!@#")

person.errors[:name]
 # => ["cannot contain the characters !@#%*()_-+="]

person.errors.full_messages
 # => ["Name cannot contain the characters !@#%*()_-+="]
```

Другой способ использования заключается в установлении `[]=`

```ruby
  class Person < ActiveRecord::Base
    def a_method_used_for_validation_purposes
      errors[:name] = "cannot contain the characters !@#%*()_-+="
    end
  end

  person = Person.create(:name => "!@#")

  person.errors[:name]
   # => ["cannot contain the characters !@#%*()_-+="]

  person.errors.to_a
   # => ["Name cannot contain the characters !@#%*()_-+="]
```

### `errors[:base]`

Можете добавлять сообщения об ошибках, которые относятся к состоянию объекта в целом, а не к отдельному аттрибуту. Этот метод можно использовать, если вы хотите сказать, что объект невалиден, независимо от значений его атрибутов. Поскольку `errors[:base]` массив, можете просто добавить строку к нему, и она будет использована как сообщение об ошибке.

```ruby
class Person < ActiveRecord::Base
  def a_method_used_for_validation_purposes
    errors[:base] << "This person is invalid because ..."
  end
end
```

### `errors.clear`

Метод `clear` используется, когда вы намеренно хотите очистить все сообщения в коллекции `errors`. Естественно, вызов `errors.clear` для невалидного объекта фактически не сделает его валидным: сейчас коллекция `errors` будет пуста, но в следующий раз, когда вы вызовете `valid?` или любой метод, который пытается сохранить этот объект в базу данных, валидации выполнятся снова. Если любая из валидаций провалится, коллекция `errors` будет заполнена снова.

```ruby
class Person < ActiveRecord::Base
  validates :name, :presence => true, :length => { :minimum => 3 }
end

person = Person.new
person.valid? # => false
person.errors[:name]
 # => ["can't be blank", "is too short (minimum is 3 characters)"]

person.errors.clear
person.errors.empty? # => true

p.save # => false

p.errors[:name]
 # => ["can't be blank", "is too short (minimum is 3 characters)"]
```

### `errors.size`

Метод `size` возвращает количество сообщений об ошибке для объекта.

```ruby
class Person < ActiveRecord::Base
  validates :name, :presence => true, :length => { :minimum => 3 }
end

person = Person.new
person.valid? # => false
person.errors.size # => 2

person = Person.new(:name => "Andrea", :email => "andrea@example.com")
person.valid? # => true
person.errors.size # => 0
```
