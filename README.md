# Анализ гипотез развития территории Красноборского сельского поселения
Этот репозиторий содержит данные, методы и результаты анализа существующего положения и трех гипотез развития территории Красноборского сельского поселения. Рассматривается оценка сценариев развития с использованием методов анализа городских данных.

## Цель исследования
Целью исследования является изучение и сравнение трех гипотез развития территории с точки зрения:
<ul>
 <li><b>Транспортной связанности и доступности:</b> оценка уровня интеграции территории в существующую транспортную сеть и расчет доступности ключевых объектов;</li>
 <li><b>Обеспеченности сервисами:</b> анализ доступности базовых и специализированных услуг для населения;</li>
 <li><b>Центральности территории:</b> оценка центральности районов на основе транспортной связанности и плотности населения в кварталах.</li>
</ul>
<p>Итогом работы является выявление наиболее жизнеспособного сценария развития территории, который способен повысить качество жизни населения и устойчивость поселения.</p>

## Команда
<ul>
 <li><b>Мартынова Олеся </b></li>
 <li><b>Шеховцов Виктор</b></li>
</ul>

## Используемы методы 
Для анализа были использованы следующие методы, представленные в библиотеке BlocksNet:
<ul>
  <li>метод вычисление связности кварталов</li>
  <li>метод подсчёт обеспеченности сервисами: остановка общественного транспорта, супермаркет, детский сад, школа</li>
  <li>метод оценки центральности по транспортной связности и разнообразию сервисов в кварталах</li>
  <li>метод оценка центральности по транспортной связности и населению в квартала</li>
</ul>

## Подготовка данных
Для анализа были собраны следующие данные по каждому сценарию, представляющие из себя геослои данных: 
<table>
  <thead>
    <tr>
      <th>Название</th>
      <th>Геометрия</th>
      <th>Атрибуты</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>boundary</td>
      <td>polygon</td>
      <td>-</td>
    </tr>
    <tr>
      <td>roads</td>
      <td>line</td>
      <td>-</td>
    </tr>
    <tr>
      <td>buildings</td>
      <td>polygon</td>
       <td>id, geometry, building, living_area, is_living, building:levels, population</td>
    </tr>
    <tr>
      <td>services:school, kindergarten, bus_station, supermarket</td>
      <td>point</td>
       <td>-</td>
    </tr>
  </tbody>
</table>
Все исходные данные геослёв хранятся в папках соответсвующих сценариев в папке <a href = "https://github.com/Dropsell/Krasnii-Bor---Analyze/tree/main/data">data</a>


## Сценарии
### Сценарий 0 - Существующее положение
<p>В нулемов сценарии рассматривается текущая ситуация в сельском поселение, где нет никакой застройки</p>
<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/0%20scenario.png" height = 500></img>

### Сценарий 1 - Сохранение ландшафта
<p>В данной гипотезы предполагается сохранение природного ландшафта на основе существующего положения</p>
<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/1%20scenario.png" height = 500></img>

### Сценарий 2 - Развитие промышленности
<p>Данная гипотеза ориентрованна на развитие промышленности</p>
<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/2%20scenario.png" height = 500></img>

### Сценарий 3 - Высокая урбанизация
<p>В данной гипотезе предполагается развитие высокоурбанизированной территории</p>
<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/3%20scenario.png" height = 500></img>

## Создание модели и реазлизация методов
Для каждого сценария был созданы файлы ```.ipynb```, расположенные в папке <a href = "https://github.com/Dropsell/Krasnii-Bor---Analyze/tree/main/dev">dev</a>. Файлы отличаются только входными данными, исполняемый код индентичен. 

1. Сначала добавляются границы территории, улично-дорожная сеть существующей застройки из файлов ```boundary.geojson```, ```roads.geojson```
2. На основе этих файлов генерируются кварталы с помощью библиотеки ```BlocksGenerator``` из BlocksNet

3. С помощью метода ```AccessibilityProcessor``` строится матрица связности. Полученные кварталы и матрица заносятся в модель
```py
city = City(
    blocks=blocks,
    acc_mx=acc_mx
)
```

4. Методом ```Connectivity``` расчитывается картограмма связности. 

<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/connection%20cartograms.png"></img>

5. Далее идёт обработка зданий из файла ```building.geojson```. Устанавливаются жилые\нежилые здания, восполняется население в жилых зданиях. Полученный слой зданий выгружается в модель.
6. Затем в модель выгружаются все файлы с сервисами, представляемые слоями с точками. Это:
<ul>
 <li>супермаркеты;</li>
 <li>школы;</li>
 <li>детсткие сады;</li>
 <li>остановки общественного транспорта.</li>
</ul>

```py
import os
import geopandas as gpd
from shapely.geometry import Point
from blocksnet import ServiceType

directory = os.fsencode(f"{data_path}/servises/")
for file in os.listdir(directory):
    filename = os.fsdecode(file)
    if filename.endswith(".geojson"):
        service_name = filename.removesuffix(".geojson")
        print(f"Adding service {service_name}")
        service_gdf = gpd.read_file(f"{data_path}/servises/" + filename)
        service_gdf.to_crs(local_crs, inplace=True)
        service = service_gdf[["geometry"]]
        try:
          city.update_services(service_name, service)
        except:
          continue
    else:
        continue
```

7.  С помощью ```Provision``` был произведёт подсчёт обеспеченности кварталов сервисами: супермаркеты, школы, детсткие сады, остановки общественного транспорта.
```py
from blocksnet import Provision, ProvisionMethod
service_type = 'bus_station'
prov = Provision(city_model=city)
prov_res = prov.calculate(service_type)
prov.plot(prov_res)
```

<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/scenarion%200%20services%20cartograms.png"></img>
<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/scenarion%201%20services%20cartograms.png"></img>
<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/scenarion%202%20services%20cartograms.png"></img>
<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/scenarion%203%20services%20cartograms.png"></img>

8.  После чего с помощью ```Centrality``` была произведена оценка центральности по транспортной связности и разнообразию сервисов в кварталах

```py
centrality = Centrality(city_model=city)
result_centrality = centrality.calculate()
result_centrality
```
<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/connection%20services%20carograms.png"></img>

9.  Далее ```PopulationCentrality```- оценка центральности по транспортной связности и населению в кварталах

```py
centrality_population = PopulationCentrality(city_model=city)
result_centrlity_population = centrality_population.calculate()
result_centrlity_population
```
<img src = "https://github.com/Dropsell/Krasnii-Bor---Analyze/blob/main/img/connection%20population%20cartograms.png"></img>


## Анализ результатов
Оценка выявления наиболее жизнеспособного сценария развития территории были применены следующие критерии оценки:
<table>
  <thead>
    <tr>
      <th>Критерий оценки</th>
      <th>Значение для нулевого сценария</th>
      <th>Значение для гипотезы №1</th>
      <th>Значение для гипотезы №2</th>
      <th>Значение для гипотезы №3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Связанность территории </td>
      <td>от 0 до 9</td>
      <td>от 0 до 8</td>
      <td>от 0 до 10</td>
      <td>от 0 до 9</td>
</tr>
   <tr>
      <td>Обеспеченность территории остановками общественного транспорта</td>
      <td>0.060</td>
      <td>0.088</td>
      <td>0.083</td>
      <td>0.110</td>
    </tr>
    <tr>
       <td>Обеспеченность территории детскими садами</td>
       <td>0.060</td>
       <td>0.035</td>
       <td>0.070</td>
       <td>0.097</td>
    </tr>
    <tr>
      <td>Обеспеченность территории школами</td>
      <td>0.060</td>
      <td>0.028</td>
      <td>0.065</td>
      <td>0.073</td>     
    </tr>
    <tr>
      <td>Обеспеченность территории супермаркетами</td>
      <td>0.003</td>
      <td>0.017</td>
      <td>0.032</td>
      <td>0.043</td>
    </tr>
    <tr>
      <td>Оценка центральности по транспортной связности и разнообразию сервисов в кварталах</td>
      <td>от 0 до 0.35</td>
      <td>от 0 до 0.5</td>
      <td>от 0 до 0.4</td>
      <td>от 0 до 0.5</td>
    </tr>
   <tr>
      <td>Оценка центральности по транспортной связности и населению в кварталах</td>
      <td>от 0 до 10</td>
      <td>от 0 до 10</td>
      <td>от 0 до 10</td>
      <td>от 0 до 10</td>
    </tr>
  </tbody>
</table>

### 

## Вывод
### Эффективность гипотез:
Сравнение гипотез развития позволяет выявить сильные и слабые стороны каждого сценария. Например, одна гипотеза может улучшать транспортную связанность, но быть недостаточной с точки зрения обеспечения сервисами.

### Рекомендации по развитию:
Итоги анализа позволяют предложить конкретные меры для реализации наиболее эффективного сценария развития. Результаты исследования формируют базу для принятия решений по развитию Красноборского сельского поселения.

### Приоритетные районы:
Определены ключевые зоны для развития, которые требуют дополнительных инвестиций в транспортную инфраструктуру и создание сервисов. Также важно равномерное распределение сервисов, чтобы сократить неравенство между центральными и удаленными районами.
