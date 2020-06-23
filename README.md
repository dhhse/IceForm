# IceForm
Formulas in Icelandic saga prose

* Презентация - https://docs.google.com/presentation/d/1pUMmJQOvvl_N9aHotdnjJlXsVhXN3Wa8rnjSFzrPvzc/edit?usp=sharing
* Презентация 02.12.2019 - https://docs.google.com/presentation/d/1LwG24KMhL-5pNuiuJjfF95VgtVStxUfoHJs6lNyHCAU/edit#slide=id.g6bd2edd304_0_17
* Корпус - http://www.malfong.is/index.php?lang=en&pg=fornritin

### Команда
Руководитель: Дарья Глебова

Участники: 
* Женя Глазунов 
* Аня Кондратьева 
* Настя Костяницына

## Описание проекта
В произведениях фольклора и литературы могут встречаться повторяющиеся конструкции. С одной стороны, это могут быть формулы - повторяющиеся формулировки, встречающиеся во многих сагах и употребляющиеся в схожих содержательных или композиционных условиях (например, формула сказочного зачина "Жили-были"). С другой стороны, повторения фраз, встречающиеся в конкретной саге - повторы часто оказываются нарративным приемом в саговом повествовании (например, в "Саге о Курином Торире" одна и та же фраза в различных вариациях встречается четыре раза, таким образом обрамляется сожжение Кетиля Сони). Цель проекта - автоматизированный (непредзаданный) поиск повторяющихся конструкций с той или иной степенью вариативности.

## Данные
Данные - Корпус древнеисландских саг <br>
Объем корпуса - 1.5 млн токенов <br>
Разметка - словоизменительный тип и лемма <br>

## Типы конструкций
Выделяют три типа конструкций: 
* <b>Закрытые</b> - лексически идентичные конструкции <br> 
  * <i>X hét maðr  (Человека звали Х)</i>

* <b>Полуоткрытые</b> - лексически разные, но семантически похожие (слова могут быть заменены на синонимы): <br>
  * <i>skiljask með kærleik (Они расстались по-дружески)</i>
  * <i>skiljask með blíðu (Они расстались по-дружески)</i>
  * <i>skiljask með vináttu (Они расстались по-дружески)</i>

* <b>Открытые</b> - схожие синтаксические конструкции, но лексически и семантически разные <br>
  * <i>Ekki hefi ek nýligra frétt en ránit (никаких новостей, кроме ограбления)</i>
  * <i>Ekki höfum vér nú nýligar frétt en brennu Blund-Ketils bónda (никаких новостей, кроме сожжения Кетиля)</i>

## Алгоритм
Несмотря на то, что при традиционной работе "руками" подобные конструкции легко заметить, их поиск занимает большое количество времени. По этой причине нам стало интересно попробовать автоматизировать процесс поиска и разработать алгоритм, который хотя бы частично облегчил задачу исследователей. 

Работа состоит из следующих этапов: 
* формирование списка нграмм
* фильтрация на основе лингвистических особенностей исландского языка 
* «схлопывание» контекстных вариантов (открытый и полуоткрытый типы конструкций)
* кластеризация
* создание базы данных
* создание сайта

В связи с тем, что некоторые варианты одной формулы могут встречаться очень редко, просто отсеивать нграммы по частотности не получается. Более того, устройство исландского языка усложняет нашу работу (свободный порядок слов, обилие компаундов и тд) и требует разработки специального алгоритма фильтрации, учитывающего особенности нашего языкового материала.

## Конструкции
Для каждой нграммы мы сохраняем следующую информацию:
* Номер текста, в котором она встретилась 
* Номер предложения
* Индекс первого слова нграммы
* Индекс последнего слова нграммы 

В рамках настоящего проекта были выделены основные критерии нграмм, на основе которых можно осуществлять их фильтрацию. Нграмма - последовательность слов, которая:
* обязательно содержит глагол
* синтаксически цельная
* имеет частеречную значимость не менее 90%

### Синтаксическая цельность
Каждое слово в корпусе сопровождается морфологической разметкой. На основе разборов каждого слова определяется морфологический состав целой нграммы для проверки синтаксической цельности  (нет нарушений согласования и управления). Например, если в нграмме есть предлог, который управляет дательным падежом, в ней обязательно должно быть существительное, местоимение или числительное в соответствующем падеже. 

### Частеречная значимость
Каждой части речи было сопоставлено число, отражающее смысловую и “градообразующую” значимость в процессе формирования фраз. Частеречная значимость нграммы рассчитывается:

- <img src="https://latex.codecogs.com/gif.latex?\frac{\sum{Significance*Frequency}}{nwords}" />

* Significance - значимость части речи
* Frequency - частотность части речи в нграмме
* length - кол-во слов в нграмме

Если полученное значение меньше 0.9, нграмма пропускается. Порог был выведен путем проверки результатов на тестовой выборке. 


## Нахождение расстояния между нграммами

Далее для того, чтобы сравнивать нграммы, они группируются по набору частей речи в очищенной нграмме. Например, все нграммы из глагола, существительного в номинативе и существительного в дативе идут в одну группу независимо от слов внутри.
Эти нграммы сравниваются внутри группы с помощью взвешенного косинусного расстояния между fasttext векторами слов.

* Нграммы записываются в таблицу, где столбцы - POS-теги (глагол, существительное в Х падеже и т.д.)
* Слова векторизуются с помощью fasttext модели от Facebook
* Вектора слов в столбце попарно сравниваются (находится косинусное расстояние)
  * если таких POS-тегов два, то слова выравниваются по лучшему совпадению. NG1 A и B, NG2 C и D. Если (A, D) = min, то A+D, B+C пары в выравнивании.
* Расстояния умножаются на коэффициент важности (глагол менять дороже). Коэффициенты примерно похожи на использованные ранее для важности
* Расстояния суммируются и получается матрица NxN с попарными расстояниями между нграммами
* Далее эта матрица расстояний используется для кластеризации.
В качестве алгоритма кластеризации выбрана иерархическая кластеризация методом полной связи. Расстояния нормализуются деление на число слов нграмме. Устанавливается трешхолд 1 (пока так). В один кластер попадают все группы, где расстояние не больше 1. Если в кластере > 1 элемента, то он сохраняется в базу.

