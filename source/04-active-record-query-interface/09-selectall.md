# select_all

У `find_by_sql` есть близкий родственник, называемый `connection#select_all`. `select_all` получит объекты из базы данных, используя произвольный SQL, как и в `find_by_sql`, но не создаст их экземпляры. Вместо этого, вы получите массив хэшей, где каждый хэш указывает на запись.

```ruby
Client.connection.select_all("SELECT * FROM clients WHERE id = '1'")
```
