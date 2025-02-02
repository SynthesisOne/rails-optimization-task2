# Case-study оптимизации

## Актуальная проблема
В нашем проекте возникла серьёзная проблема.

Необходимо было обработать файл с данными, чуть больше ста мегабайт.

У нас уже была программа на `ruby`, которая умела делать нужную обработку.

Она успешно работала на файлах размером пару мегабайт, но для большого файла она работала слишком долго, и не было понятно, закончит ли она вообще работу за какое-то разумное время.

Я решил исправить эту проблему, оптимизировав эту программу.

## Формирование метрики
Для того, чтобы понимать, дают ли мои изменения положительный эффект на быстродействие программы я придумал использовать такую метрику: потребление памяти

## Гарантия корректности работы оптимизированной программы
Программа поставлялась с тестом. Выполнение этого теста в фидбек-лупе позволяет не допустить изменения логики программы при оптимизации.

## Feedback-Loop
Для того, чтобы иметь возможность быстро проверять гипотезы я выстроил эффективный `feedback-loop`, который позволил мне получать обратную связь по эффективности сделанных изменений за ~ 3 секунд

Вот как я построил `feedback_loop`: я создал файл на 10 тысяч строк для быстрой проверки результата моих изменений к коде

## Вникаем в детали системы, чтобы найти главные точки роста
Для того, чтобы найти "точки роста" для оптимизации я в основном использовал memory_profiler и stackprof

Вот какие проблемы удалось найти и решить


### Ваша находка №1
Обновил Ruby до версии 3.0.0
изменений не замечено

### Ваша находка №2
Использовал jemalloc
изменений не замечено

### Ваша находка №3
MALLOC_ARENA_MAX=2
изменений не замечено

### Ваша находка №4
- по совету из readme решил перевести программу на потоковое выполнение(честно пришлось подсмотреть у других ребят как реализована технически потоковая запись файла(не хватило времени) но концепт я понимал)
- valgrind показал что за все время исполнения скрипта после перевода на потокое чтение файла
 на пике использовалось не более ~41.3мб памяти что укладывается в бюджет

## Результаты
В результате проделанной оптимизации наконец удалось обработать файл с данными.
Удалось улучшить метрику системы с 500мб до ~41.3мб и уложиться в заданный бюджет.
Использовал все инструменты показанные в скринкастах и теперь хочу применить это на рабочем проекте где есть проблемы с памятью

бонусом стало что время исполнения скрипта уменьшилось вдвое, и по сути скрипт теперь способен переварить любой объем данных

## Защита от регрессии производительности
Для защиты от потери достигнутого прогресса при дальнейших изменениях программы были обновлены тесты по скорости исполнения
и написан тест для проверки памяти до выполнения и после выполнения скрипта и их разница
