# Mire

Это ненасильственная МПД. (Многопользовательское подземелье)

## Usage

Сначала убедитесь, что на вашем компьютере установлена `java`
. [OpenJDK](https://adoptopenjdk.net ) рекомендуется. Это должна
быть как минимум версия 8, но более новые версии (протестированные до 17) тоже должны работать.

Выполните команду "./lein run" внутри каталога Mire, чтобы запустить сервер Mire
. Затем игроки могут подключиться по telnet к порту 3333.

## Motivation

Основная цель этой кодовой базы - демонстрация того, как
создать простой многопоточный сервер в Clojure.

Mire создается шаг за шагом, где каждый шаг вводит один или два
небольших, но ключевых принципа Clojure и основывается на последнем шаге.
Каждый из этих шагов существует в отдельных ветвях git. Чтобы получить максимальную отдачу от
чтения Mire, вам следует начать чтение в ветке под названием
[step-01-echo-server](http://github.com/technomancy/mire/tree/01-echo-server )
и продолжайте оттуда.

Хотя вы можете учиться у Mire сами по себе, она была написана
специально для [скринкаста PluralSight на
Clojure](https://www.pluralsight.com/courses/functional-programming-clojure ).
[Запись в блоге](https://technomancy.us/136 ) пошагово описывает кодовую
базу и показывает, как внести незначительные обновления для более свежей версии Clojure.

Авторское право © 2009-2021 Фил Хагельберг
Лицензируется на тех же условиях, что и Clojure.

# Наш Проект

## Над доработкой проекта работали:

1. Красношеин Даниил
2. Тиунов Никита
3. Хлыбов Дмитрий

## Как запустить

1. Убедитесь в том, что у вас установлен Linux, либо есть виртуальная машина на ОС Linux
2. Необходимо склонировать данный репозиторий на свой ПК `git clone`
3. Для запуска игры со стороны Clojure, обратите внимание на раздел **Usage**
4. Для запуска бота, необходимо установить приложение SWI-Prolog
5. По кнопке file-Consult... подключиться к файлу Bot.pl
6. в консоли Prolog написать `main.`

## Как играть 

Попадая в игру, вам будет предложено ввести свое имя (вводите, что хотите, главное чтобы это был текст или цифры). Затем вам высветится характеристика вашего персонажа и вы попадете в первую комнату (будьте аккуратны и не пугайтесь XD). В комнате вам будет предложено к выбору несколько направлений, где находится следующая комната. Чтобы пойти куда-либо, необходимо выбрать направление и написать одну из команд в консоли `north`, `west`, `south`, `east`. Также вам будут встречаться предметы, их можно подобрать командой `grab *название предмета*`.

## Добавленный функционал

В первую очередь, мы расширили функционал игрока, добавив несколько новых характеристик:

```clojure
(def ^:dynamic *player-name*)
(def ^:dynamic *health*)
(def ^:dynamic *score*)
(def ^:dynamic *status*)
(def ^:dynamic *money*)
(def ^:dynamic *armor*)
(def ^:dynamic *weapon*)
(def ^:dynamic *last-message*)
```

Для реализации новых характеристик игрока, нам потребовалось расширить предметы в комнатах:

```clojure
(defn load-room [rooms file]
  (let [room (read-string (slurp (.getAbsolutePath file)))]
    (conj rooms
          {(keyword (.getName file))
           {:money (ref (:money (or (:items room))))
            :weapons (ref (or (:weapons (:items room)) #{}))
            :armors (ref (or (:armors (:items room)) #{})
```

Было решено добавить несколько комнат, а так же понадобилось переписать функционал изначальных комнат под новые предметы:

### closet
```clojure
{:desc "This is the Armor Store. And yes, it's closet."
 :exits {:south :start}
 :items {:money 1 :weapons #{} :armors #{}}
 :store {:armor {:divine 1 :magic 2 :heavy 3}}}
```

### restroom
'''clojure
{:desc "It's our little hostpital."
 :exits {:north :start :south :bedroom :east :promenade :west :promenade}
 :items {:money 7 :weapons #{} :armors #{}}}
```

### bedroom
```clojure
{:desc "Maybe you wanna watch TV?"
 :exits {:north :restroom :south :kitchen}
 :items {:money 7 :weapons #{} :armors #{}}}
```

И переписали server.clj под нашу реализацию.

```clojure
(defn load-room [rooms file]
  (let [room (read-string (slurp (.getAbsolutePath file)))]
    (conj rooms
          {(keyword (.getName file))
           {:money (ref (:money (or (:items room))))
            :weapons (ref (or (:weapons (:items room)) #{}))
            :armors (ref (or (:armors (:items room)) #{})
```

Самым сложной и трудоёмкой работой - вышла реализация бота, файл -> mirebot.pl. Было реализовано: взаимодействие с игровым сервером, обработка команд перемещения, взятия оружия и брони, отправка сообщений и запрос статистики.
