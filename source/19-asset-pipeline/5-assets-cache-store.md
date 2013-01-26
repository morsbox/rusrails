# Хранилище кэша ресурсов

Sprockets использует хранилище кэша Rails по умолчанию для кэширования ресурсов в development и production. Это может быть изменено настройкой `config.assets.cache_store`.

```ruby
config.assets.cache_store = :memory_store
```

Опции, принимаемые хранилищем кэша ресурсов, те же самые, что и для хранилища кэша приложения.

```ruby
config.assets.cache_store = :memory_store, { :size => 32.megabytes }
```
