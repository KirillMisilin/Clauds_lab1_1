# Лабораторная 1*  
## Цель работы  
Приложение в запущенном контейнере должно записывать изменения в базу данных. Что именно записать – передаётся в команде запуска контейнера. При остановке контейнера информация не должна исчезать.
Файл должен иметь название, отличное от Dockerfile. (то есть докер файл должен называться не Dockerfile).  
## Ход работы  
Для выполнения работы выбран проект на Django.  
Django имеет встроенную базу данных SQLLite, поэтому запускать отдельно базу данных нет необходимости.  
Dockerfile переименован в Dockerfile_good. Содержимое файла:  
```
FROM python:3 AS builder
EXPOSE 8000
WORKDIR /first_django
COPY requirements.txt /first_django
RUN pip3 install -r requirements.txt --no-cache-dir
COPY . /first_django
ENTRYPOINT ["python3"]
RUN python3 manage.py shell < script.py
CMD ["manage.py", "runserver", "0.0.0.0:8000"]
```  
Запись в БД производится с помощью команды, запускающей скрипт в оболочке Django:  
```
RUN python3 manage.py shell < script.py
```  
Код скрипта:  
```
from django_app1.models import Character
Character(name="test3").save()
```
Скрипт сохраняет запись с именем test3 в базу данных.  

Для сборки контейнера используется файл compose.yaml:  
```
services:
  web: 
    build:
      context: first_django
      target: builder
      dockerfile: Dockerfile_good
    ports:
      - '8000:8000'
```
Для использования нестандартного имени Dockerfile оно должно быть указано в строке dockerfile:  
```
      dockerfile: Dockerfile_good  
```
## Тестирование  
Сборка успешна:  
![Снимок экрана (2614)](https://github.com/KirillMisilin/Clauds_lab1_1/assets/88585791/7ba1d5ba-99d0-4ca7-8ce9-37f9b45f0230)  
![Снимок экрана (2615)](https://github.com/KirillMisilin/Clauds_lab1_1/assets/88585791/b7a01511-cc35-42ed-bdfb-49b55a93bf33)  
После запуска контейнера появилась новая запись в БД:  
![Снимок экрана (2616)](https://github.com/KirillMisilin/Clauds_lab1_1/assets/88585791/1e967880-8d9a-4805-86d5-79a52d795333)  
После остановки контейнера и повторного запуска запись не исчезла.
