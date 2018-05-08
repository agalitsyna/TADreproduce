# TADreproduce
Воспроизводим https://bitbucket.org/mforcato/hictoolscompare/src

## Мысли
Хочу меру схожести границ ТАДов, чтобы сравнить результаты программ с симулированными данными. Так, для каждой гаммы будет набор JI по всем симуляциям, надо строить распределение (либо бокс-плоты, если гамм мало - для арматуса, либо брать среднее/медиану и строить линию, если гамм много - modulation_score в lavaburst). Так сможем сравнивать границы ТАДов с идеальными из симуляций и между собой. Тогда надо написать функцию, которая умеет считать JI и OC для двух наборов границ ТАДов (двумерных массивов np.array)

Мера схожести границ ТАДов - JI и OC. Функции есть в файле HiCToolsCompare / TADs / 00_utils_tads.r, надо перевести в питон.

compare_tads_with_string_approach - непонятно, что делает.
calculate_concordance - считает JI и OC, пишет в файл.

Анализ симуляций в файле HiCToolsCompare / Simulations / Analyze_simulations_TADs.R Там сравнивают обнаруженные ТАДы и границы ТАДов обычным in. Функция np.isin(array1, array2) возвращает логический массив, который можно инвертировать с помощью ~.

## Имеем следующие объекты:
- Сколько-то методов (данные по x)
- Управляющий параметр метода (данные по x)
- 25 различных симуляций (данные по x, генерация выборки)
- Шум в матрице контактов (данные по x)
- Границы получающихся ТАДов (сырые результаты, генерация выборки)
- Размер ТАДов (данные по y)
- Число ТАДов (данные по y)
- JI и OC (данные по y)
- TPR и FDR (данные по y)

## Задача для симулированных данных
Надо повторить suppl fig 13, добавить подбор управляющего параметра. Найти лучшую гамму в зависимости от FDR, TPR, ROC-curve, для каждого метода в зависимости от шума и для каждой симуляции.

## Работа с симуляцией:
- Получить ТАДы для нескольких значений управляющего параметра. Сделать так для нескольких методов для каждой симуляции. Перевести все файлы в один формат (два столбца - начало и конец тадов). Сохранить информацию о шуме, о симуляции, методе и управляющем параметре в названии файла. Формат: метод_гамма_шум_номер.txt
- ROC-curve для каждого метода в зависимости от гаммы, найти лучшую гамму исходя из этой кривой. Сделать для разных значений шума и для всех симуляций в совокупности. Мб построить в 3D такой график. Сделать как для самих ТАДов, так и для их границ. (наша новинка)
- Посмотреть на число тадов в зависимости от шума для каждого метода для лучшей гаммы и для всех гамм.
- Размер тадов для каждого метода в репликах, учитывать шум. Для лучшей гаммы и для всех гамм.
- TPR, FDR в зависимости от шума для каждого метода c наилучшей гаммой (это наша новинка)

## Методы
### Умеем делать
- Armatus, гамма от 0 до 5 с шагом в 0.5, работает медленно, надо на кластере
- lavaburst: armatus_score (от 0 до 5 с шагом в 0.5), modularity_score (от 0 до 100 с шагом в 1), potts_score (), variance_score (). Можно сделать у себя на компе, работает довольно быстро.
- HiCseg, nb_change_max - наибольшее число точек смены (число ТАДов - 1?). Работает медленно, жрёт много памяти, надо на кластере. Судя по всему, не имеет управляющего параметра.

### Что ещё предлагают
- Arrowhead - написан на java, есть ли управляющий параметр? Конвертация результатов, по-видимому, не нужна.
- domainCaller - странная штука, неохота разбираться.
- TADtree - слишком сложно, много параметров и какая-то непонятная конверсия результатов.
- insulationScore - на перле, просто, надо сделать. ! Не заводится, медленный.
- Tadtool - на него подаётся матрица и файл с координатами бинов. Имеет insulation score, directionality index. https://github.com/vaquerizaslab/tadtool

- TADbit - выглядит несложно, управляющий параметр?
- HiCExplorer - новый метод, нет в статье, выглядит безобидно, но использует какой-то свой формат матрицы контактов, надо конвертировать. В качестве входного формата принимает npz - он получается при сохранении np.array с помощью np.savez. Возможно, dekker - это подходящий формат. По ссылке как превратить обычную матрицу ndarray в npz: https://github.com/deeptools/HiCExplorer/issues/50

Уже есть armatus и несколько из lavaburst, пока стоит в очереди HiCseg. Надо сделать InsulationScore, arrowhead, TADbit, DomainCaller, TADtree в порядке снижения приоритета и повышения сложности.

