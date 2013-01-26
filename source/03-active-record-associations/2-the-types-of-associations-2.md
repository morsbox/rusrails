# Типы связей (часть вторая)

Продолжение. [Часть первая тут](/active-record-associations/the-types-of-associations-1).

### Выбор между `belongs_to` и `has_one`

Если хотите настроить отношение один-к-одному между двумя моделями, необходимо добавить `belongs_to` к одной и `has_one` к другой. Как узнать что к какой?

Различие в том, где помещен внешний ключ (он должен быть в таблице для класса, объявляющего связь `belongs_to`), но вы также должны думать о реальном значении данных. Отношение `has_one` говорит, что что-то принадлежит вам - то есть что что-то указывает на вас. Например, больше смысла в том, что поставщик владеет аккаунтом, чем в том, что аккаунт владеет поставщиком. Это означает, что правильные отношения подобны этому:

```ruby
class Supplier < ActiveRecord::Base
  has_one :account
end

class Account < ActiveRecord::Base
  belongs_to :supplier
end
```

Соответствующая миграция может выглядеть так:

```ruby
class CreateSuppliers < ActiveRecord::Migration
  def change
    create_table :suppliers do |t|
      t.string  :name
      t.timestamps
    end

    create_table :accounts do |t|
      t.integer :supplier_id
      t.string  :account_number
      t.timestamps
    end
  end
end
```

NOTE: Использование `t.integer :supplier_id` указывает имя внешнего ключа очевидно и явно. В современных версиях Rails можно абстрагироваться от деталей реализации используя `t.references :supplier`.

### Выбор между `has_many :through` и `has_and_belongs_to_many`

Rails предлагает два разных способа объявления отношения многие-ко-многим между моделями. Простейший способ - использовать `has_and_belongs_to_many`, который позволяет создать связь напрямую:

```ruby
class Assembly < ActiveRecord::Base
  has_and_belongs_to_many :parts
end

class Part < ActiveRecord::Base
  has_and_belongs_to_many :assemblies
end
```

Второй способ объявить отношение многие-ко-многим - использование `has_many :through`. Это осуществляет связь не напрямую, а через соединяющую модель:

```ruby
class Assembly < ActiveRecord::Base
  has_many :manifests
  has_many :parts, :through => :manifests
end

class Manifest < ActiveRecord::Base
  belongs_to :assembly
  belongs_to :part
end

class Part < ActiveRecord::Base
  has_many :manifests
  has_many :assemblies, :through => :manifests
end
```

Простейший признак того, что нужно настраивать отношение `has_many :through` - если необходимо работать с моделью отношений как с независимым объектом. Если вам не нужно ничего делать с моделью отношений, проще настроить связь `has_and_belongs_to_many` (хотя нужно не забыть создать соединяющую таблицу в базе данных).

Вы должны использовать `has_many :through`, если нужны валидации, колбэки или дополнительные атрибуты для соединительной модели.

### Полиморфные связи

_Полиморфные связи_ - это немного более "навороченный" вид связей. С полиморфными связями модель может принадлежать более чем одной модели, на одиночной связи. Например, имеется модель изображения, которая принадлежит или модели работника, или модели продукта. Вот как это объявляется:

```ruby
class Picture < ActiveRecord::Base
  belongs_to :imageable, :polymorphic => true
end

class Employee < ActiveRecord::Base
  has_many :pictures, :as => :imageable
end

class Product < ActiveRecord::Base
  has_many :pictures, :as => :imageable
end
```

Можете считать полиморфное объявление `belongs_to` как настройку интерфейса, который может использовать любая другая модель. Из экземпляра модели `Employee` можно получить коллекцию изображений: `@employee.pictures`.

Подобным образом можно получить `@product.pictures`.

Если имеется экземпляр модели `Picture`, можно получить его родителя посредством `@picture.imageable`. Чтобы это работало, необходимо объявить столбец внешнего ключа и столбец типа в модели, объявляющей полиморфный интерфейс:

```ruby
class CreatePictures < ActiveRecord::Migration
  def change
    create_table :pictures do |t|
      t.string  :name
      t.integer :imageable_id
      t.string  :imageable_type
      t.timestamps
    end
  end
end
```

Эта миграция может быть упрощена при использовании формы `t.references`:

```ruby
class CreatePictures < ActiveRecord::Migration
  def change
    create_table :pictures do |t|
      t.string :name
      t.references :imageable, :polymorphic => true
      t.timestamps
    end
  end
end
```

![Диаграмма для полиморфной связи](/assets/guides/polymorphic.png)

### Самоприсоединение

При разработке модели данных иногда находится модель, которая может иметь отношение сама к себе. Например, мы хотим хранить всех работников в одной модели базы данных, но нам нужно отслеживать отношения начальник-подчиненный. Эта ситуация может быть смоделирована с помощью самоприсоединяемых связей:

```ruby
class Employee < ActiveRecord::Base
  has_many :subordinates, :class_name => "Employee",
    :foreign_key => "manager_id"
  belongs_to :manager, :class_name => "Employee"
end
```

С такой настройкой, вы можете получить `@employee.subordinates` и `@employee.manager`.
