# Расширения для Range

### `to_s`

Active Support расширяет метод `Range#to_s` так, что он понимает необязательный аргумент формата. В настоящий момент имеется только один поддерживаемый формат, отличный от дефолтного, это `:db`:

```ruby
(Date.today..Date.tomorrow).to_s
# => "2009-10-25..2009-10-26"

(Date.today..Date.tomorrow).to_s(:db)
# => "BETWEEN '2009-10-25' AND '2009-10-26'"
```

Как изображено в примере, формат `:db` создает SQL условие `BETWEEN`. Это используется Active Record в его поддержке интервальных значений в условиях.

NOTE: Определено в `active_support/core_ext/range/conversions.rb`.

### `include?`

Методы `Range#include?` и `Range#===` говорит, лежит ли некоторое значение между концами заданного экземпляра:

```ruby
(2..3).include?(Math::E) # => true
```

Active Support расширяет эти методы так, что аргумент может также быть другим интервалом. В этом случае тестируется, принадлежат ли концы аргумента самому получателю:

```ruby
(1..10).include?(3..7)  # => true
(1..10).include?(0..7)  # => false
(1..10).include?(3..11) # => false
(1...9).include?(3..9)  # => false
(1..10) === (3..7)  # => true
(1..10) === (0..7)  # => false
(1..10) === (3..11) # => false
(1...9) === (3..9)  # => false
```

NOTE: Определено в `active_support/core_ext/range/include_range.rb`.

### `overlaps?`

Метод `Range#overlaps?` говорит, имеют ли два заданных интервала непустое пересечение:

```ruby
(1..10).overlaps?(7..11)  # => true
(1..10).overlaps?(0..7)   # => true
(1..10).overlaps?(11..27) # => false
```

NOTE: Определено в `active_support/core_ext/range/overlaps.rb`.
