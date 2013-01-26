# Другие подходы к тестированию

Тестирование, основанное на встренном `test/unit`, не является единственным способом тестировать приложение на Rails. Разработчики на Rails прибегают к различным подходам и вспомогательным инструментам для тестирования, включающим:

* [NullDB](http://avdi.org/projects/nulldb/), способ ускорить тестирование, избегая использование базы данных.
* [Factory Girl](https://github.com/thoughtbot/factory_girl/tree/master), замена для фикстур.
* [Machinist](https://github.com/notahat/machinist/tree/master), другая замена для фикстур.
* [Shoulda](http://www.thoughtbot.com/projects/shoulda), расширение для `test/unit` с дополнительными хелперами, макросами и операторами контроля.
* [RSpec](http://relishapp.com/rspec), фреймворк разработки, основанной на поведении
