# Расширения для Time

### Вычисления

NOTE: Все следующие методы определены в `active_support/core_ext/date_time/calculations.rb`.

Active Support добавляет к `Time` множество методов, доступных для `DateTime`:

```ruby
past?
today?
future?
yesterday
tomorrow
seconds_since_midnight
change
advance
ago
since (in)
beginning_of_day (midnight, at_midnight, at_beginning_of_day)
end_of_day
beginning_of_hour (at_beginning_of_hour)
end_of_hour
beginning_of_week (at_beginning_of_week)
end_of_week (at_end_of_week)
monday
sunday
weeks_ago
prev_week (last_week)
next_week
months_ago
months_since
beginning_of_month (at_beginning_of_month)
end_of_month (at_end_of_month)
prev_month (last_month)
next_month
beginning_of_quarter (at_beginning_of_quarter)
end_of_quarter (at_end_of_quarter)
beginning_of_year (at_beginning_of_year)
end_of_year (at_end_of_year)
years_ago
years_since
prev_year (last_year)
next_year
```

Это аналоги. Обратитесь к их документации в предыдущих разделах, но примите во внимание следующие различия:

* `change` принимает дополнительную опцию `:usec`.
* `Time` понимает летнее время (DST), поэтому вы получите правильные вычисления времени как тут:

```ruby
Time.zone_default
# => #<ActiveSupport::TimeZone:0x7f73654d4f38 @utc_offset=nil, @name="Madrid", ...>

# В Барселоне, 2010/03/28 02:00 +0100 становится 2010/03/28 03:00 +0200 благодаря переходу на летнее время.
t = Time.local_time(2010, 3, 28, 1, 59, 59)
# => Sun Mar 28 01:59:59 +0100 2010
t.advance(:seconds => 1)
# => Sun Mar 28 03:00:00 +0200 2010
```

* Если `since` или `ago` перепрыгивает на время, которое не может быть выражено с помощью `Time`, вместо него возвращается объект `DateTime`.

#### `Time.current`

Active Support определяет `Time.current` как сегодняшний день в текущей временной зоне. Он похож на `Time.now`, за исключением того, что он учитывает временную зону пользователя, если она определена. Он также определяет `Time.yesterday` и `Time.tomorrow`, и условия экзеппляра `past?`, `today?` и `future?`, все они относительны к `Time.current`.

При осуществлении сравнения Time с использованием методов, учитывающих временную зону пользователя, убедитесь, что используете `Time.current`, а не `Time.now`. Есть случаи, когда временная зона пользователя может быть в будущем по сравнению с временной зоной системы, в которой по умолчанию используется `Time.today`. Это означает, что `Time.now` может быть равным `Time.yesterday`.

#### `all_day`, `all_week`, `all_month`, `all_quarter` и `all_year`

Метод `all_day` возвращает интервал, представляющий целый день для текущего времени.

```ruby
now = Time.current
# => Mon, 09 Aug 2010 23:20:05 UTC +00:00
now.all_day
# => Mon, 09 Aug 2010 00:00:00 UTC +00:00..Mon, 09 Aug 2010 23:59:59 UTC +00:00
```

Аналогично `all_week`, `all_month`, `all_quarter` и `all_year` служат целям создания временных интервалов.

```ruby
now = Time.current
# => Mon, 09 Aug 2010 23:20:05 UTC +00:00
now.all_week
# => Mon, 09 Aug 2010 00:00:00 UTC +00:00..Sun, 15 Aug 2010 23:59:59 UTC +00:00
now.all_month
# => Sat, 01 Aug 2010 00:00:00 UTC +00:00..Tue, 31 Aug 2010 23:59:59 UTC +00:00
now.all_quarter
# => Thu, 01 Jul 2010 00:00:00 UTC +00:00..Thu, 30 Sep 2010 23:59:59 UTC +00:00
now.all_year
# => Fri, 01 Jan 2010 00:00:00 UTC +00:00..Fri, 31 Dec 2010 23:59:59 UTC +00:00
```

### Конструкторы Time

Active Support определяет `Time.current` как `Time.zone.now`, если у пользователя определена временная зона, а иначе `Time.now`:

```ruby
Time.zone_default
# => #<ActiveSupport::TimeZone:0x7f73654d4f38 @utc_offset=nil, @name="Madrid", ...>
Time.current
# => Fri, 06 Aug 2010 17:11:58 CEST `02:00
```

Как и у `DateTime`, условия `past?` и `future?` выполняются относительно `Time.current`.

Используйте метод класса `local_time`, чтобы создать объекты времени, учитывающие временную зону пользователя:

```ruby
Time.zone_default
# => #<ActiveSupport::TimeZone:0x7f73654d4f38 @utc_offset=nil, @name="Madrid", ...>
Time.local_time(2010, 8, 15)
# => Sun Aug 15 00:00:00 +0200 2010
```

Метод класса `utc_time` возвращает время в UTC:

```ruby
Time.zone_default
# => #<ActiveSupport::TimeZone:0x7f73654d4f38 @utc_offset=nil, @name="Madrid", ...>
Time.utc_time(2010, 8, 15)
# => Sun Aug 15 00:00:00 UTC 2010
```

И `local_time`, и `utc_time` принимают до семи позиционных аргументов: year, month, day, hour, min, sec, usec. Year обязателен, month и day принимаются по умолчанию как 1, остальное по умолчанию 0.

Если время, подлежащее конструированию лежит за рамками, поддерживаемыми `Time` на запущенной платформе, usecs отбрасываются и вместо этого возвращается объект `DateTime`.

#### Длительности

Длительности могут быть добавлены и вычтены из объектов времени:

```ruby
now = Time.current
# => Mon, 09 Aug 2010 23:20:05 UTC +00:00
now ` 1.year
#  => Tue, 09 Aug 2011 23:21:11 UTC +00:00
now - 1.week
# => Mon, 02 Aug 2010 23:21:11 UTC +00:00
```

Это преводится в вызовы `since` или `advance`. Для примера выполним корректный переход во время календарной реформы:

```ruby
Time.utc_time(1582, 10, 3) + 5.days
# => Mon Oct 18 00:00:00 UTC 1582
```
