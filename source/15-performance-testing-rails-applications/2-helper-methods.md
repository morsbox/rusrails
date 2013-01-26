# Методы хелпера

Rails предоставляет различные методы хелпера в Active Record, Action Controller и Action View для измерения времени, затраченного на заданный кусок кода. Метод называется `benchmark()` во всех трех компонентах.

h4. Модель

```ruby
Project.benchmark("Creating project") do
  project = Project.create("name" => "stuff")
  project.create_manager("name" => "David")
  project.milestones << Milestone.all
end
```

Это произведет бенчмаркинг кода, заключенного в блок `Project.benchmark("Creating project") do...end` и напечатает результат в файл лога:

```ruby
Creating project (185.3ms)
```

Пожалуйста, обратитесь к [API docs](http://api.rubyonrails.org/classes/ActiveSupport/Benchmarkable.html#method-i-benchmark), чтобы узнать дополнительные опции для `benchmark()`

### Контроллер

Подобным образом можно использовать этот метод хелпера в [контроллерах](http://api.rubyonrails.org/classes/ActiveSupport/Benchmarkable.html)

```ruby
def process_projects
  benchmark("Processing projects") do
    Project.process(params[:project_ids])
    Project.update_cached_projects
  end
end
```

NOTE: `benchmark` это метод класса в контроллерах.

### Вьюха

И во [вьюхах](http://api.rubyonrails.org/classes/ActiveSupport/Benchmarkable.html):

```erb
<% benchmark("Showing projects partial") do %>
  <%= render @projects %>
<% end %>
```
