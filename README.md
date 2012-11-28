yaml_temp
=========

Пояснения по поводу YAML
## Общие моменты
### 1) Символ "-" (без кавычек) образует массив. Нужно внимательно отнестись к ним:
```YAML
- test1
- test2
- test3
```
образует
```Ruby
["test1","test2","test3"]
```

### 2) "ключ: значение" образует хэш:
```YAML
name: Это имя
value: Это значение
```
образует
```Ruby
{"name" => "Это имя", "value" => "Это значение"}
```
но если переписать чуть по другому пример:
```YAML
- name: Это имя
- value: Это значение
```
то объект в Ruby будет выглядеть уже совсем по другому. Это будет массив хэшей:
```Ruby
[{"name" => "Это имя"},{"value" => "Это значение"}]
```

### 3) Очень важны отступы. Например если нам нужно определить несколько значений для ключа напрмиер values:
```YAML
name: Это имя
attrs:
  attr1: Значение 1
  attr2: Значение2
  attr3: Третье значение
```
то в Ruby это будет выглядеть как:
```Ruby
{"name"=>"Это имя", "attrs"=>{"attr1"=>"Значение 1", "attr2"=>"Значение2", "attr3"=>"Третье значение"}}
```
но если убрать отступы:
```YAML
name: Это имя
attrs:
attr1: Значение 1
attr2: Значение2
attr3: Третье значение
```
то получится совсем другой код:
```Ruby
{"name"=>"Это имя", "attrs"=>nil, "attr1"=>"Значение 1", "attr2"=>"Значение2", "attr3"=>"Третье значение"}
```
**Важно**: Используйте 2 пробела в качестве отступа. Sublime Text 2 можно настроить что бы он воспринимал 
нажатие TAB как 2 пробела:
```Text
Preferences -> Settings - Default
Установить "tab_size": 2
Установить "translate_tabs_to_spaces": true
```
## Формат для импорта
### 1) Синонимы
* Любое значение может использовать синонимы
* Синонимы пишутся через исмвол "@" (at, без ковычек)

Например у нас есть Категория (товарная группа):

```YAML
- name: Кирпич облицовочный силикатный@Кирпич облицовочный безобжиговый метод
  attributes:
    - name: Состав
      values:
        - Силикатный с кварцевым песком
        - Силикатный известково-шлаковый
        - Силикатный известково-зольный
    - name: Вид кирпича
      values:
        - Полнотелый
        - Пустотелый
    - name: Фактура поверхности граней
      values:
        - Гладкий
        - Рельефный
        - Рустированный@со сколотой фактурой
    - name: Размер
      values:
      - КО@кирпич одинарный
      - КЕ@кирпич «ЕВРО»
      - КУ@кирпич утолщенный полуторный
      - КМ@кирпич модульный одинарный 
      - КМУ@кирпич модульный утолщенный
    - name: Марка@прочность
      values:
        - M25
        - M35
        - M50
        - M75
        - M100
        - M125
        - M150
    - name: Морозостойкость
      values:
        - 25
        - 35
        - 50
        - 75
        - 100
      metric: F@циклы
    - name: Средняя плотность
      values:
        - Заполняется производителем
      metric: кг/м3
    - name: Коэфф. теплопроводности@Коэффициент теплопроводности@Теплопроводность
      values:
        - Заполняется производителем
    - name: Цвет
      values:
        - Заполняется производителем
    - name: Фактурные поверхности
      values:
        - Ложковая
        - Тычковая  
```

Как видно из кода у некоторых значений есть синонимы, например есть аттрибут 
с **name** => **"Коэфф. теплопроводности@Коэффициент теплопроводности@Теплопроводность"**.

### 2) Регулярные выражения в синонимах

Допустим надо написать вместо двух синонимов названия аттрибута **"Коэфф. теплопроводности"** один шаблон который бы 
включал их оба:

```YAML
- name: Коэфф. теплопроводности@/.*теплопроводност.*/i
```
Что означает это выржаение:
```Ruby 
/.*теплопроводност.*/i 
```
Оно означает:
```Text
.* - любой символ 0 или более раз
i  - не учитывать регистр букв (трочные ли прописные)
Получается что выражение совпадет со любой строчкой в которой в любом месте встретится "теплопроводност", или же
"Теплопроводост" или же "ТеПЛОпровоДност" и т.д.

```
Проверяем что получилось в Ruby:
```Ruby
1.9.3p286 :073 > /.*теплопроводност.*/i.match("Коэфф. теплопроводности")
 => #<MatchData "Коэфф. теплопроводности"> 
1.9.3p286 :074 > /.*теплопроводност.*/i.match("Теплопроводность")
 => #<MatchData "Теплопроводность"> 
1.9.3p286 :075 > /.*теплопроводност.*/i.match("Коэффициент теплопроводности")
 => #<MatchData "Коэффициент теплопроводности"> 
```
Возьмем дргой пример с аттритбутом **Марка**
```YAML
- name: Марка@прочность
  values:
    - M25
    - M35
    - M50
    - M75
    - M100
    - M125
    - M150
```

Одно из возможных значений - **M100**. Но от поставщика может прийти в таких вариантах **M 100**, **M,100**, **M-100** 
или **m100**. При этом с шаблоном не должны совпадать например **"SM100"** или **"M/100"** или **" M-asdfsdf-100"**
Пишем шаблон:
```YAML
/(^|\s+)M[^\/]{0,1}100/i
```
разеберем выражение:
```Text
(^|\s+) - совпадает либо с началом строки либо с одним или более пробелом. Выражение взято в скобки для того что бы 
          иметь возможность поставить "или". Детальный разбор:
  ()    - группировка выражения что бы можно было поставить "или"
  ^     - начало строки
  \s    - пробел (или табуляция, в общем пустой символ)
  +     - один или более
  |     - или

M       - буква М

[^\/]{0,1}  - совпадет с любым символом кроме символа "/". Детальный разбор:
  []    - квадратные скобки означают что в них будет набор явно указанных символов
  ^     - в квадратных скобках означает что должно НЕ совпасть с указанными символами (отрицание)
  \/    - означает символ "/". Для того что бы Ruby не воспринял его как окончание регулярного выражения, 
          перед ним ставится спец символ "\"
  {0,1} - 0 или 1 раз

100     - символы 100

i       - модификатор означающий что не надо учитывать регистр символов

Таким образом наше выражение означает:
Найти любую подстроку которая 
 * стоит либо в начале строки либо имеет перед собой один или более пробелов,
 * начинается с буквы M,
 * после буквы M содержит любой символ кроме символа "/" 0 или 1 раз
 * после всего вышеописанного содержит символы 100
```

Проверяем в Ruby
```Ruby
1.9.3p286 :077 > /(^|\s+)M[^\/]{0,1}100/i.match("M 100")
 => #<MatchData "M 100" 1:""> 
1.9.3p286 :078 > /(^|\s+)M[^\/]{0,1}100/i.match("M100")
 => #<MatchData "M100" 1:""> 
1.9.3p286 :079 > /(^|\s+)M[^\/]{0,1}100/i.match("M/100")
 => nil 
1.9.3p286 :080 > /(^|\s+)M[^\/]{0,1}100/i.match("SM100")
 => nil 
1.9.3p286 :081 > /(^|\s+)M[^\/]{0,1}100/i.match("M,100")
 => #<MatchData "M,100" 1:""> 
1.9.3p286 :082 > /(^|\s+)M[^\/]{0,1}100/i.match("M-100")
 => #<MatchData "M-100" 1:""> 
1.9.3p286 :083 > /(^|\s+)M[^\/]{0,1}100/i.match("Это один раз M-100 или другой раз M100")
 => #<MatchData " M-100 или другой раз M100" 1:" "> 
```

Таким образом исходный аттрибут можно переписать так:

```YAML
- name: Марка@прочность
  values:
    - M25
    - M35
    - M50
    - M75
    - "M100@/(^|\s+)M[^\/]{0,1}100/i"
    - M125
    - M150
```

## Ресурсы
Общие сведения о реглярных выражениях
http://ru.wikipedia.org/wiki/%D0%A0%D0%B5%D0%B3%D1%83%D0%BB%D1%8F%D1%80%D0%BD%D1%8B%D0%B5_%D0%B2%D1%8B%D1%80%D0%B0%D0%B6%D0%B5%D0%BD%D0%B8%D1%8F

Валидация вашего YAML файла:
http://yaml-online-parser.appspot.com/

Тестирование регулярных выражений онлайн
http://rubular.com/