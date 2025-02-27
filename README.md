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

- MAU - 3.3млрд
- DAU - 0.8 * 3.3млрд = 2.64млрд
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
Также учтем, что в среднем голосовые звонки длятся 10 минут [[5]], а видео звонки - 18 минут в день
Таким образом:\
***RPS голосовых звонков*** = 100 млн * 0.7 / (24ч * 60 мин * 60 сек) = 2'025 зап/сек\
***RPS видео звонков*** = 100 млн * 0.3 / (24ч * 60 мин * 60 сек) = 868 зап/сек\
В дальнейшем будем считать, что звонки происходят P2P и не будет учитывать в метриках.

Рассчитаем запросы на регистрацию. На основе отчета [[1]], видно, что за 2024 год
MAU увеличилось с 3млрд до 3,3 млрд. Таким образом, можно предположить, что количество новых
пользователей приблизительно 300 млн\
***RPS регистрации*** = 300 млн / (365 дней * 24ч * 60 мин * 60 сек) = 9 зап/сек

Рассчитаем запросы на поиск какого либо сообщения в чате. В среднем, активный пользователь использует функцию 
поиска 2 раза в день. DAU = 2.64 млрд.
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

Рассчитаем RPS на чтение новых сообщений. Будем оперировать числами полученными выше, так как сообщений отправленных
всегда больше чем полученных. Кроме того, учтем, что в групповых чатах читают сообщений сразу несколько людей. 
Согласно отчету [[10]] в личных чатах дольше суток остаются непрочитанными примерно 20% сообщений, остальные сообщения
прочитываются в течение часа. В групповых же чатах примерно 50% сообщений остаются непрочитанными хотя бы одним пользователем
в то время как остальные 50% читаются в течение часа. Будем считать, что оставшиеся 50% читает только половина пользователей чата. 
Тогда имеем следующие RPS на чтение в личном чате:\
***RPS чтения текстовых сообщений в личном чате*** = 497'664 зап/сек * 0.8 = 398'131 зап/сек\
***RPS чтения голосовых сообщений в личном чате*** = 34'841 зап/сек * 0.8 = 27'872 зап/сек\
***RPS чтения сообщений с фото в личном чате*** = 199'074 зап/сек * 0.8 = 159'259 зап/сек\
***RPS чтения сообщений с видео в личном чате*** = 99'537 зап/сек * 0.8 = 79'485 зап/сек\
***RPS чтения сообщений с иными файлами в личном чате*** =49'769 зап/сек * 0.8 = 39'815 зап/сек

На чтение в групповом чате имеем:\
***RPS чтения текстовых сообщений в групповом чате*** = 659'693 * 0.5 * 30 + 659'693 * 0.5 * 15 = 14'843'092зап/сек\
***RPS чтения голосовых сообщений в групповом чате*** = 46'183 * 0.5 * 30 + 46'183 * 0.5 * 15 = 1'039'117зап/сек\
***RPS чтения сообщений с фото в групповом чате*** = 263'888 * 0.5 * 30 + 263'888 * 0.5 * 15 = 5'937'480зап/сек\
***RPS чтения сообщений с видео в групповом чате*** = 131'944 * 0.5 * 30 + 131'944 * 0.5 * 15 = 2'968'740 зап/сек\
***RPS чтения сообщений с иными файлами в групповом чате*** = 65'971 * 0.5 * 30 + 65'971 * 0.5 * 15 = 1'484'347 зап/сек

Рассчитаем RPS на создание нового чата. Выше выяснили, что количество новых пользователей за 2024 год - 
300млн. Каждый пользователь участвует в 15-20 групповых чатах [[7]], а в среднем в одном чате состоит 30 пользователей. 
Таким образом:\
***Количество новых чатов за год*** = 20 * 300млн / 30 = 200млн\
***RPS создания новых чатов*** = 200млн / (365 дней * 24ч * 60 мин * 60 сек) = 6 зап/сек

Кроме того, нужно учесть что расчеты приведены для средних значений RPS, нужно понимать, что бывают часы пиковых нагрузок. 
Для таких случаев введем коэффициент запаса = 2

### Расчёт трафика
Большая часть сообщений, согласно исследованиям [8] не превышает 10 слов. Так как наиболее используемый и общий известный
язык английский, то проверим среднюю длину слова именно на это языке. Согласно исследованию [9] средняя длина слова 5 символов.
Стандартная кодировка UTF8 занимает максимум 4 байта информации, таким образом размер текстового сообщения:\
10 слов * 5 букв * 4 байта = 200 байт
Кроме того, нужно помнить, что в сообщении присутствуют метаданные, например ID отправителя, время, статус. Заложим для 
этого еще некоторое количество байт, пусть размер сообщения будет 512 байт = 0.5 Кбайт. 
***Трафик текстовых сообщений в личных чатах*** = 497'664 зап/сек * 0.5 Кбайт = 248'822 Кбайт/с * 8 / 1024 / 1024 = 1.8 ГБита/с\
***Трафик текстовых сообщений в групповых чатах*** = 659'693 зап/сек * 0.5 Кбайт = 329'846 Кбайт/с * 8 / 1024 / 1024 = 2.5 ГБита/с

Среднее время голосового сообщения примем равным 30 секунд, а битрейт 320 Кбит/с - как наивысшее качество кодирования, тогда трафик ГС:\
***Трафик голосовых сообщений в личном чате*** = 34'841 * 320 * 30 / 1024 / 1024 = 319 Гбита/с\
***Трафик голосовых сообщений в групповом чате*** = 46'183 * 320 * 30 / 1024 / 1024 = 423 Гбита/с

Рассчитаем средний размер трафика с файлами. Средний размер фотографий на современных смартфонах 4Мбайт.
Пусть в среднем отправляются видео длиной 1 минута, учтем также, что большинство пользователей whatsapp, как было представлено
на графике в первом пункте из бедных стран, видео на телефонах будут сниматься в качестве hd 30. Но 15% пользователей используют 
более развитые смартфоны и снимают видео в среднем с разрешением 4к 30, тогда средний размер видео в 4к 30: 200Мбайт, а в hd 30: 60Мбайт
остальные файлы пусть имеют средний размер
PDF-файлов - 10Мбайт, таким образом получаем:\
***Трафик сообщений с фото в личных чатах*** = 199'074 зап/сек * 4Мб * 8 / 1024 = 6'221 ГБита/с\
***Трафик сообщений с фото в групповых чатах*** = 263'888 зап/сек * 4Мб * 8 / 1024 = 8'246 ГБита/с\
***Трафик сообщений с видео в личных чатах*** = 99'537 зап/сек * (200 * 0.15 + 60 * 0.85) Мбайт * 8 / 1024 / 1024 = 60 ТБита/с\
***Трафик сообщений с видео в групповых чатах*** = 131'944 зап/сек * (200 * 0.15 + 60 * 0.85) Мбайт * 8 / 1024 / 1024 = 81 ТБита/с\
***Трафик сообщений с иными файлами в личных чатах*** = 49'769 зап/сек * 10 Мбайт * 8 / 1024 = 3'888 ГБита/с\
***Трафик сообщений с иными файлами в групповых чатах*** = 65'971 зап/сек * 10 Мбайт * 8 / 1024 = 5'154 ГБита/с

Теперь рассчитаем аналогичный трафик для чтения всех видов сообщений:\
***Трафик чтения текстовых сообщений в личных чатах*** = 398'131 зап/сек * 0.5 Кбайт = 248'822 Кбайт/с * 8 / 1024 / 1024 = 1.5 ГБита/с\
***Трафик чтения текстовых сообщений в групповых чатах*** = 14'843'092 зап/сек * 0.5 Кбайт = 329'846 Кбайт/с * 8 / 1024 / 1024 = 57 ГБита/с
***Трафик чтения голосовых сообщений в личном чате*** = 27'872 * 320 * 30 / 1024 / 1024 = 255 Гбита/с\
***Трафик чтения голосовых сообщений в групповом чате*** = 1'039'117 * 320 * 30 / 1024 / 1024 / 1024 = 9.3 Тбита/с
***Трафик чтения сообщений с фото в личных чатах*** = 159'259 зап/сек * 4Мб * 8 / 1024 / 1024 = 4'9 ТБита/с\
***Трафик чтения сообщений с фото в групповых чатах*** = 5'937'480 зап/сек * 4Мб * 8 / 1024 / 1024 = 182 ТБита/с\
***Трафик чтения сообщений с видео в личных чатах*** = 79'485 зап/сек * (200 * 0.15 + 60 * 0.85) Мбайт * 8 / 1024 / 1024 = 49 ТБита/с\
***Трафик чтения сообщений с видео в групповых чатах*** = 2'968'740 зап/сек * (200 * 0.15 + 60 * 0.85) Мбайт * 8 / 1024 / 1024 = 1'845 ТБита/с\
***Трафик чтения сообщений с иными файлами в личных чатах*** = 39'815 зап/сек * 10 Мбайт * 8 / 1024 = 3'110 ГБита/с\
***Трафик чтения сообщений с иными файлами в групповых чатах*** = 65'971 зап/сек * 10 Мбайт * 8 / 1024 / 1024 = 113.2 ТБита/с

Рассчитаем трафик поиска сообщений по чату. Обычно поиск выполняется по одному ключевому слову, средний размер слова рассчитывали выше
Таким образом:\
***Трафик поиска по чату*** = 61'111 зап/сек * (5 букв * 4 байта + 300 байт) * 8 бит / 1024 / 1024 / 1024 = 0.15 Гбит/с 

Рассчитаем трафик на создание групповых чатов. Максимальная длина названия чата в whatsapp 25 символом, каждый кодируется 
4 байтами + метаинформация, примерно 100 байт. Кроме того средний чат содержит 30 пользователей, следовательно, отправляется
информация об этих пользователях (ID), 30 * 4 = 120 байт. Таким образом:\
***Трафик создания групповых чатов*** = 6 * (120 + 100 + 4 * 25) байт * 8 бит / 1024 = 15Кбит/с.\
Этот трафик довольно мал по сравнению с остальными, следовательно, не будем его учитывать и аппроксимируем 0 в таблице

Рассчитаем трафик на регистрацию. Для регистрации необходимо имя, номер телефона и аватарка. Пусть аватарка весит 1Мб, тогда
весом имени и номера телефона можно учесть как метаданные. С учетом необходимости всех метаданных примем размер профиля
1.3Мб. Тогда:\
***Трафик регистрации*** = 1.3 * 9 * 8 / 1024 = 0.1Гбит/с


### Расчет хранилища
Необходимо хранить: новые сообщения, новые группы и новых пользователей. Также учтем, что сообщения хранятся на сервере 
лишь до того времени, пока пользователь их не прочтет. Остальное время они хранятся на клиенте. Кроме того сообщения хранятся 
один месяц. Согласно отчету [[10]] в личных чатах дольше суток остаются непрочитанными примерно 20% сообщений, остальные сообщения
прочитываются в течение часа. В групповых же чатах примерно 50% сообщений остаются непрочитанными хотя бы одним пользователем
в то время как остальные 50% читаются в течение часа. 

***Файловое хранилище постоянная основа ежедневно*** = ((319+6'221+150'000+3'888) * 0.2 + (422+8'246+201'000+5'154) * 0.5) * (24ч * 60 минут * 60 секунд) / 1024 / 1024 / 8 = 1'437 Пбайт\
***Файловое хранилище в течение часа*** = ((319+6'221+150'000+3'888) * 0.8 + (422+8'246+201'000+5'154) * 0.5) *(60 минут * 60 секунд) / 1024 / 1024 / 8 = 101Пбайт 

Таким образом необходимое хранилище, с учетом хранения в течение месяца:\
***Файловое хранилище*** = 1'437 * 30 + 101 = 43'211 Пбайт

Информацию о новых пользователях и группах храним постоянно, сообщения же хранятся так же один месяц, в случае их не прочтения
Таким образом основное хранилище:\
***Основное хранилище текст не читаемый*** = (1.8 * 0.2 + 2.5 * 0.5) * (24ч * 60 минут * 60 секунд) * 30 дней / 1024 / 1024 / 8 = 0.5Пбайт/
***Основное хранилище текст читаемый*** = (1.8 * 0.8 + 2.5 * 0.5) * (60 минут * 60 секунд) / 1024 / 1024 / 8 = 0.001Пбайт

***Основное хранилище пользователи и группы*** = (3.3 млрд * 1.3МБ + 0.11 млрд * 0.001МБ) / 1024 / 1024 / 1024= 4Пбайт

***Основное хранилище*** = 4Пб + 0.001Пб + 0.5Пб = 4.6 Пбайт.

Рассчитаем потенциальный рост хранилищ. Основное хранилище будет расти за счет увеличения текстовых сообщений + создание
новых пользователей и новых групп.\
***Основное хранилище пользователи и группы рост*** = (0.1 + 0.00002) * 365 дней * 24ч * 60 минут * 60 секунд / 1024 / 1024 / 8 = 0.4ПБайт в год

Исходя из приведенных выше данных получаем, что в среднем, каждый пользователь отправляет 30 сообщения в день. Таким образом
рост хранилища сообщений за год:\
***Основное хранилище сообщений рост*** = 9 * 365 дней * 24ч * 60 минут * 60 секунд * 30 * 512 байт / 1024 / 1024 / 1024 / 1024 / 1024 = 0.004Пбайт в год\

***Основное хранилище рост в год*** = 0.4+0.004 = 0.45 Пбайт/год

Аналогично исходя из приведенных выше данных получаем, что в среднем, каждый пользователь отправляет 2 голосовых сообщения в день\
***Рост хранилища из-за ГС*** = 9 * 365 дней * 24ч * 60 минут * 60 секунд * 2 * 320 * 30 / 8 / 1024 / 1024 / 1024 / 1024 = 0.62 Пбайт/год

Рассчитаем рост хранилища файлов из-за отправленных вложений, как было сказано раньше фото - 20%, видео - 10%, остальные типы файлов - 5%. 
Таким образом, имеем:\
***Рост хранилища из-за видео*** = 9 * 365 * 24 * 60 * 60 * 30 * 0.1 * 200Мбайт / 1024 / 1024 / 1024 = 159Пбайт в год\
***Рост хранилища из-за фото*** = 9 * 365 * 24 * 60 * 60 * 30 * 0.2 * 4Мбайт / 1024 / 1024 / 1024 = 6.4Пбайт в год\
***Рост хранилища из-за иных файлов*** = 9 * 365 * 24 * 60 * 60 * 30 * 0.05 * 10Мбайт / 1024 / 1024 / 1024 = 4Пбайт в год\

***Рост хранилища файлов в год*** = 159 + 6.4 + 4 + 0.62 = 170 Пбайт/год

Кроме того на все хранилище нужно ввести коэффициент запаса, так как в процессе расчета были округления и возможно всплески
прибытия новых пользователей, заложим под это дополнительно 15%.

## Результаты расчетов
|                       Метрика                       |    RPS     | RPS пиковое | Трафик, Гб/с | Трафик пиковое, Гб/с |
|:---------------------------------------------------:|:----------:|:-----------:|:------------:|:--------------------:|
|     Отправка текстовых сообщений в личном чате      |  497'664   |   995'328   |     1.8      |         3.6          |
|    Отправка текстовых сообщений в групповом чате    |  659'693   |  1'319'386  |     2.5      |          5           |
|     Отправка голосовых сообщений в личном чате      |   34'841   |   69'682    |     319      |         638          |
|    Отправка голосовых сообщений в групповом чате    |   46'183   |   92'366    |     422      |         844          |
|       Отправка сообщений с фото в личном чате       |  199'074   |   398'148   |    6'221     |        12'442        |
|     Отправка сообщений с фото в групповом чате      |  263'888   |   527'776   |    8'246     |        16'292        |
|      Отправка сообщений с видео в личном чате       |   99'537   |   199'074   |    60'000    |       120'000        |
|    Отправка  сообщений с видео в групповом чате     |  131'944   |   263'888   |    81'000    |       161'000        |
|  Отправка сообщений с иными файлами в личном чате   |   49'769   |   99'538    |    3'888     |        7'776         |
| Отправка сообщений с иными файлами в групповом чате |   65'971   |   131'942   |    5'154     |        10'308        |
|               Поиск сообщений по чату               |   61'111   |   122'222   |     0.15     |         0.3          |
|              Создание групповых чатов               |     6      |     12      |      0       |          0           |
|                     Регистрация                     |     9      |     18      |     0.1      |         0.2          |
|      Чтение текстовых сообщений в личном чате       |  398'131   |   796'262   |     1.5      |          3           |
|     Чтение текстовых сообщений в групповом чате     | 14'843'092 | 29'686'184  |      57      |         104          |
|      Чтение голосовых сообщений в личном чате       |   27'872   |   55'744    |     255      |         510          |
|     Чтение голосовых сообщений в групповом чате     | 1'039'117  |  2'078'235  |    9'300     |        18'600        |
|        Чтение сообщений с фото в личном чате        |  159'259   |   318'518   |    4'900     |        9'800         |
|      Чтение сообщений с фото в групповом чате       | 5'937'480  | 11'874'960  |   182'000    |       364'000        |
|       Чтение сообщений с видео в личном чате        |   79'485   |   158'970   |    49'000    |        98'000        |
|     Чтение  сообщений с видео в групповом чате      | 2'968'740  |  5'937'480  |  1'845'000   |      3'690'000       |
|   Чтение сообщений с иными файлами в личном чате    |   39'815   |   79'630    |    3'110     |        6'220         |
|  Чтение сообщений с иными файлами в групповом чате  | 1'484'347  |  2'968'694  |   113'200    |       226'400        |


|       Метрика хранилища        |   Значение    | Значение с коэффициентом запаса |
|:------------------------------:|:-------------:|:-------------------------------:|
|       Файловое хранилище       |  43'211Пбайт  |           50'000Пбайт           |
|       Основное хранилище       |   4,6Пбайт    |            5.3Пбайт             |
| Рост основного хранилища в год | 0.45Пбайт/год |          0.52Пбайт/год          |
| Рост файлового хранилища в год | 170Пбайт/год  |          200Пбайт/год           |

***

[1]: https://www.coolest-gadgets.com/whatsapp-statistics/
[2]: https://learn.rasayel.io/en/blog/whatsapp-user-statistics/
[3]: https://aitechtonic.com/whatsapp-statistics/
[4]: https://www.cisco.com/c/en/us/solutions/collateral/executive-perspectives/annual-internet-report/white-paper-c11-741490.html
[5]: https://www.wharftt.com/how-long-can-a-whatsapp-call-last/
[6]: https://www.sandvine.com/hubfs/Sandvine_Redesign_2019/Downloads/2023/reports/Sandvine%20GIPR%202023.pdf
[7]: https://www.statista.com/topics/2018/whatsapp/ 
[8]: https://www.researchgate.net/publication/299487660_WhatsApp_Usage_Patterns_and_Prediction_Models
[9]: https://norvig.com/mayzner.html
[10]: https://www.pewresearch.org/internet/2021/08/19/mobile-technology-and-home-broadcast-2021/

- # Список источников
1. [Статистика и трафик Whatsapp][1]
2. [Пользовательская статистика][2]
3. [Количество пользователей Whatsapp][3]
4. [Статистика интернет-трафика][4]
5. [Среднее время звонков][5]
6. [Среднее время видео звонков][6]
7. [Количество пользователей в чатах][7]
8. [Анализ длину сообщений][8]
9. [Анализ английских слов][9]
10. [Игнорирование сообщений в личных чатах][10]


