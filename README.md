# Проектирование высоконагруженного сервиса Whatsapp
***

# Содержание
1. [Тема, MVP, целевая аудитория](#1-тема-mvp-анализ-трафика)
2. [Расчет нагрузки](#2-расчет-нагрузки)
- [Список источников](#список-источников)

***

# 1. Тема, MVP, анализ трафика
***Whatsapp*** - мессенджер, где каждый может общаться с другими людьми как один на один, так и в группе - с помощью текстовых и голосовых сообщений, а также аудио- и видео звонков.

### MVP
1. Регистрация и авторизация по номеру телефона
2. Отправка сообщений
3. Отправка вложений, в том числе голосовых сообщений
4. Просмотр истории чата
5. Поиск по чату
6. Групповые чаты
7. Аудио- и видео звонки

### Ключевые продуктовые решения
Сообщения хранятся только на клиенте. Сообщения хранятся на сервере до 30 дней в зашифрованном виде, если их не удалось
доставить сразу. В случае, если сообщение не доставлено за 30 дней, то оно удаляется с сервера.


### Анализ трафика
- MAU - 3.3 млрд [[1]]
- DAU - >80% пользователей открывают ежедневно [[2]]
- Средняя продолжительности посещения в день - 33 минуты [[2]]
- Количество сообщений в день - 100 млрд [[2]]
- Количество голосовых сообщений в день - 7 млрд [[1]]
- Количество аудио- и видео звонков в день - 100 млн [[1]] 
- Количество сообщений, отправляемых в групповых чатах - 57% [[1]]
- Количество пользователей в группе - от 10 до 50 человек, в зависимости от региона(в восточных странах группы обычно больших размеров)

#### Демографические показатели
![Демография](img/Демография.png) [[1]]

![Гендер](img/Гендер.png) [[1]]

#### Использование Whatsapp по странам
Топ - 10 стран по количеству пользователей [[3]]

| Страна    | Количество пользователей, млн |
|-----------|:-----------------------------:|
| Индия     |              535              |
| Бразилия  |              148              |
| Индонезия |              112              |
| США       |              98               |
| Филиппины |              88               |
| Мексика   |              77               |
| Россия    |              66               |
| Турция    |              56               |
| Египет    |              55               |
| Пакистан  |              52               |

#### Трафик через 5 ключевых стран [[1]]
| Страна    | Процент трафика | Всего устройств, млн | Мобильных устройств | Web/Пк |
|-----------|:---------------:|:--------------------:|:-------------------:|:------:|
| Индия     |      17.94      |         300          |       62.81%        | 37.19% |
| Бразилия  |      13.36      |         224          |       39.87%        | 60.13% |
| Индонезия |       5.6       |         93.9         |       73.43%        | 26.57% |
| США       |      4.96       |         83.2         |        29.59        | 70.41% |
| Мексика   |        4        |         67.7         |       44.07%        | 55.93% |

Мессенджер запрещён в 6 странах: [[1]]
- Китай
- ОАЭ
- Иран
- Сирия 
- Северная Корея
- Куба

***
# 2. Расчет нагрузки

### Расчёт RPS
Как было сказано выше количество сообщений, отправляемых в день - 100 млрд, таким образом:\
***RPS отправки текстовых сообщений*** = 100 млрд /(24ч * 60мин * 60сек) = 1'157'357 зап/сек

Количество голосовых сообщений также приведено выше - 7млрд, таким образом:\
***RPS отправки голосовых сообщений*** = 7млрд /(24ч * 60 мин * 60 сек) = 81'024 зап/сек

Компания Meta не раскрывает количество сообщений, которые содержат фото, видео или иные файлы в своих сообщениях.
Для оценки воспользуемся данными с сайта[[4]], там сказано, что медиафайлы составляют 60-70% трафика, но только 
25-35% от общего количество сообщений. Рассчитаем RPS с учетом того, что 35% - это медиа файлы, из них фото - 20%, 
видео - 10%, остальные типы файлов - 5%. Также учтем, что обычно при отправке медиафайлов происходит 2 запроса - 
первый POST-запрос загрузки файла на сервер, второй запрос - отправка метаданных.\
***RPS отправки сообщений с фото*** = 100млрд * 0.2 * 2 /(24ч * 60 мин * 60 сек) = 462'962 зап/сек\
***RPS отправки сообщений с видео*** = 100млрд * 0.1 * 2 /(24ч * 60 мин * 60 сек) = 231'481 зап/сек\
***RPS отправки сообщений с иными файлами*** = 100млрд * 0.05 * 2 /(24ч * 60 мин * 60 сек) = 115'740 зап/сек

Количество аудио и видеозвонков также было приведено выше и составляет - 100млн в день. 
Там учитывается и аудио и видео звонки, следовательно, для корректного расчета необходимо разделить их
в верном процентном соотношении. Обращаясь к источнику [[4]] можно найти информацию о том, что 
в среднем видео звонки составляют 30-35% от общего числа вызовов, в то время как голосовые - 65-70%.
Также учтем, что в среднем голосовые звонки длятся 10 минут [[5]], а видео звонки - 18 минут
Таким образом:\
***RPS голосовых звонков*** = 100 млн * 0.7 / (24ч * 60 мин * 60 сек) = 2'025 зап/сек\
***RPS видео звонков*** = 100 млн * 0.3 / (24ч * 60 мин * 60 сек) = 868 зап/сек

Рассчитаем запросы на регистрацию. На основе отчета [[1]], видно, что за 2024 год
MAU увеличилось с 3млрд до 3,3 млрд. Таким образом, можно предположить, что количество новых
пользователей приблизительно 300 млн\
***RPS регистрации*** = 300 млн / (365 дней * 24ч * 60 мин * 60 сек) = 9 зап/сек

Рассчитаем запросы на поиск какого либо сообщения в чате. В среднем, активный пользователь использует функцию 
поиска 2 раза в день. DAU = 80%MAU. Таким образом суточное количество активных пользователей whatsapp:\
***DAU*** = 0.8 * 3.3млрд = 2.64 млрд.\
***RPS поиска*** = 2.64 млрд * 2 / (24ч * 60 мин * 60 сек) = 61'111 зап/сек

Учтем, что анализ RPS для сообщений был приведен общий, но 57% из всех отправленных сообщений относятся к
групповым чатам. Таким образом, получаем\
***RPS отправки текстовых сообщений в групповом чате*** = 1.157.357 зап/сек * 0.57 = 659'693 зап/сек\
***RPS отправки голосовых сообщений в групповом чате*** = 81.024 зап/сек * 0.57 = 46'183 зап/сек\
***RPS отправки сообщений с фото в групповом чате*** = 462.962 зап/сек * 0.57 = 263'888 зап/сек\
***RPS отправки сообщений с видео в групповом чате*** = 231.481 зап/сек * 0.57 = 131'944 зап/сек\
***RPS отправки сообщений с иными файлами в групповом чате*** = 115.740 зап/сек * 0.57 = 65'971 зап/сек

Аналогично получаем RPS для переписок один на один\
***RPS отправки текстовых сообщений в личном чате*** = 1.157.357 зап/сек * 0.43 = 497'664 зап/сек\
***RPS отправки голосовых сообщений в личном чате*** = 81.024 зап/сек * 0.43 = 34'841 зап/сек\
***RPS отправки сообщений с фото в личном чате*** = 462.962 зап/сек * 0.43 = 199'074 зап/сек\
***RPS отправки сообщений с видео в личном чате*** = 231.481 зап/сек * 0.43 = 99'537 зап/сек\
***RPS отправки сообщений с иными файлами в личном чате*** = 115.740 зап/сек * 0.43 = 49'769 зап/сек

Рассчитаем RPS на создание нового чата. Выше выяснили, что количество новых пользователей за 2024 год - 
300млн. Каждый пользователь участвует в 15-20 групповых чатах [[7]], а в среднем в одном чате состоит 30 пользователей. 
Таким образом:\
***Количество новых чатов за год*** = 20 * 300млн / 30 = 200млн\
***RPS создания новых чатов*** = 200млн / (365 дней * 24ч * 60 мин * 60 сек) = 6 зап/сек

Кроме того, нужно учесть что расчеты приведены для средних значений RPS, нужно понимать, что бывают часы пиковых нагрузок. 
Для таких случаев введем коэффициент запаса = 2.5

### Расчёт трафика

## Результаты расчетов
|                       Метрика                       |   RPS   | RPS пиковое | Трафик, Гб/с | Трафик пиковое, Гб/с |
|:---------------------------------------------------:|:-------:|:-----------:|:------------:|:--------------------:|
|     Отправка текстовых сообщений в личном чате      | 497'664 |  1'244'160  |      0       |          0           |
|    Отправка текстовых сообщений в групповом чате    | 659'693 |  1'649'232  |      0       |          0           |
|     Отправка голосовых сообщений в личном чате      | 34'841  |   87'102    |      0       |          0           |
|    Отправка голосовых сообщений в групповом чате    | 46'183  |   115'457   |      0       |          0           |
|       Отправка сообщений с фото в личном чате       | 199'074 |   497'685   |      0       |          0           |
|     Отправка сообщений с фото в групповом чате      | 263'888 |   659'720   |      0       |          0           |
|      Отправка сообщений с видео в личном чате       | 99'537  |   248'842   |      0       |          0           |
|    Отправка  сообщений с видео в групповом чате     | 131'944 |   329'860   |      0       |          0           |
|  Отправка сообщений с иными файлами в личном чате   | 49'769  |   124'422   |      0       |          0           |
| Отправка сообщений с иными файлами в групповом чате | 65'971  |   164'927   |      0       |          0           |
|               Поиск сообщений по чату               | 61'111  |   152'777   |      0       |          0           |
|              Создание групповых чатов               |    6    |     15      |      0       |          0           |
|                  Голосовые звонки                   |  2'025  |    5'062    |      0       |          0           |
|                    Видео звонки                     |   868   |    2'170    |      0       |          0           |
|                     Регистрация                     |    9    |     23      |      0       |          0           |


***

[1]: https://www.coolest-gadgets.com/whatsapp-statistics/
[2]: https://learn.rasayel.io/en/blog/whatsapp-user-statistics/
[3]: https://aitechtonic.com/whatsapp-statistics/
[4]: https://www.cisco.com/c/en/us/solutions/collateral/executive-perspectives/annual-internet-report/white-paper-c11-741490.html
[5]: https://www.wharftt.com/how-long-can-a-whatsapp-call-last/
[6]: https://www.sandvine.com/hubfs/Sandvine_Redesign_2019/Downloads/2023/reports/Sandvine%20GIPR%202023.pdf
[7]: https://www.statista.com/topics/2018/whatsapp/ 

- # Список источников
1. [Статистика и трафик Whatsapp][1]
2. [Пользовательская статистика][2]
3. [Количество пользователей Whatsapp][3]
4. [Статистика интернет-трафика][4]
5. [Среднее время звонков][5]
6. [Среднее время видео звонков][6]
7. [Количество пользователей в чатах][7]


