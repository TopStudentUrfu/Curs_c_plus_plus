# Лекция 13: Паттерны проектирования  
**Паттерн** — структурное решение для типовой проблемы проектирования.  
**Преимущества:**  
- Готовые продуманные решения  
- Учтены нюансы реализации  
- Снижение риска ошибок  

---
### 1. задача -  реальзовать возможность сьедания фруктов различными способами:
**Требования:**  
1. Способы съедания:  
   - Проглотить целиком (`swallow`)  
   - Откусывать по частям (`bite off`)  
   - Разрезать на кусочки (`chop`)  
2. Запрещено менять интерфейс фруктов  


## Решение "в лоб" (без использования паттернов)

1. Объявить в интерфейсе фрукта функцию, которая задает "способ съедания" в виде нового перечисляемого типа ``enum class EatingMannerEnum``.
2. Дополнить свойства класса, добавив туда заданный "способ съедания": ``EatingMannerEnum EatingManner``
3. Единообразно переписать **в каждом подклассе** функцию ``void Eat()`` так, чтобы она использовала свойство EatingManner для определения того, что надо делать в процессе съедания фрукта.


- не удобное в реализации и дальнейшем изменении


## **1. Реализация паттерна "Стратегия"**
**Суть:** Инкапсуляция алгоритмов в отдельные классы.

### Решение с помощью паттерна стратегия
1. Объявить перечисляемый тип способа съедания.
2. Создать класс абстрактный класс стратегии съедания ``EatingStrategy``
3. Унаследовать от него конкретные классы стратегий: ``SwallowEatingStrategy``, ``BiteOffEatingStrategy``, ``ChopEatingStrategy``
4. Разработать фабричный метод для создания экземпляров классов стратегий съедания.
5. Инкапсулировать стратегию съедания в родительском классе ``Fruit`` с использованием указателя.
6. Дополнить интерфейс класса ``Fruit`` функцией для задания стратегии ``SetEatingManner(EatingMannerEnum)``.
7. Переписать функцию ``Eat()`` родительского класса ``Fruit``, чтобы она использовала указанную стратегию.


### Реализация:
**1. Перечисление способов:**  
```cpp
enum class EatingMannerEnum : int {
    Swallow,
    BiteOff,
    Chop,
    None
};
```

**2. Абстрактная стратегия:**  
```cpp
class EatingStrategy {
public:
    virtual ~EatingStrategy() {}
    virtual void Eat() = 0;
};
```

**3. Конкретные стратегии:**
  ```cpp
  class SwallowEatingStrategy : public EatingStrategy
  {
      void Eat() {cout << "Swallow the whole fruit"; }
  };

  class BiteOffEatingStrategy : public EatingStrategy
  {
      void Eat() {cout << "Bite off a piece..."; }
  };

  class ChopEatingStrategy : public EatingStrategy
  {
      void Eat() {cout << "Chop into little piece..."; }
  };
  ```

**4. Фабрика для создания стратегий:**
```cpp
EatingStrategy *CreateEatingStrategy(EatingMannerEnum manner)
{
    switch(manner)
    {
        case EatingMannerEnum::Swallow: return new SwallowEatingStrategy;
        case EatingMannerEnum::Biteoff: return new BiteOffEatingStrategy;
        case EatingMannerEnum::Chop: return new ChopEatingStrategy;
        default: return nullptr;
    }
}
```


**5. Интеграция во фрукты:**
Инкапсулирвоание функции класса в одном методе:
```cpp
class Fruit {
private:
    EatingStrategy *EatingManner; // Указатель на стратегию

    void DoEatUsingStrategy()
    {
      if(EatingManner == nullptr)
      {
        // Способ съедания не задан, ничего не делаем
        cout << "Do nothing!";
        return;
      }
      else
      {
        // Съесть заданным способом
        EatingManner->Eat();
      }
    }

};
```

---
**5.2. начинаем редактировать сами классы**
```cpp
class Fruit {
private:
    FruitColor Color;
    double Weight;
    EatingStrategy *EatingManner; // Указатель на стратегию

protected:
    bool FruitIsGood;

public:
    Fruit(FruitColor color) : Color(color), Weight(0.0), FruitIsGood(false), 
                              EatingManner(nullptr) {
        FruitIsGood = static_cast<bool>(rand()%2);
    }

    virtual ~Fruit() {
        if(EatingManner != nullptr) delete EatingManner; // Освобождаем память
    }

    // ... остальные методы остаются без изменений ...
};
```   
**6. Метод для установки стратегии**
```cpp
class Fruit {
    // ... предыдущий код ...

public:
    // Добавляем метод для установки стратегии
    void SetEatingManner(EatingStrategy *eatingManner) {
        EatingManner = eatingManner;
    }

    virtual void PrintType() = 0;

    virtual void Peel() = 0;
};
``` 
**6.2. добавляем в конструкторы классов:**
```cpp
Apple::Apple() : Fruit(FruitColor::Green)
{
    SetEatingManner(CreateEatingStrategy(EatingMannerEnum::BiteOff)); 
}
Kiwi() : Fruit(FruitColor::Green) 
    {
        SetEatingManner(CreateEatingStrategy(EatingMannerEnum::Swallow)); 
    }
Orange() : Fruit(FruitColor::Orange) { 
        SetEatingManner(CreateEatingStrategy(EatingMannerEnum::Chop));
    }
```

**7. Модификация метода Eat() для использования стратегии**
```cpp
class Fruit {
public:
    virtual void Eat()
    {
        if(IsGood())
        {
            cout << "Eating GOOD fruit... ";
        }
        else
        {
            cout << "Eating BAD fruit... ";
        }
        DoEatingUsingManner();
    }
};

```

**Пример: если хотим чтобы все были одинаковые:**
```cpp
ArrayClass<Fruit*> fruitArray;
for(size_t i=0; i<N; i++)
{
    int fruit_num = rand()%3+1; // Число от 1 до 3 (случайный фрукт)
    FruitType fruit_type = static_cast<FruitType>(fruit_num);
    Fruit *newFruit = CreateFruit(fruit_type);
    newFruit->SetEatingManner(CreateEatingStrategy(EatingMannerEnum::Chop)); 
    fruitArray.Add(newFruit);
}
```
---


## Краткая характеристика паттерна "Стратегия"

**Паттерн "Стратегия"** определяет семейство алгоритмов, инкапсулирует каждый из них и обеспечивает их взаимозаменяемость. Он позволяет модифицировать алгоритмы независимо от их использования на стороне клиента.

**Где надо внести правки, чтобы был еще один способ:**
- в перечеслении, класс сделать для него и еще один фабричный метод

### Преимущества "Стратегии":  
1. **Гибкость:** Легкая замена алгоритмов  
2. **Расширяемость:** Добавление новых стратегий без изменения кода фруктов  
3. **Инкапсуляция:** Каждый алгоритм в своем классе  




---

## 2 паттерн - (Шаблонный метод) 

### задача: 
1. Унифицировать структуру алгоритма метода ``Eat()`` родительского класса ``Fruit`` и унаследованных от него классов.
2. Уменьшить количество дублируемого кода в методах ``Eat()``.

### Решение с использованием паттерна "Шаблонный метод"

1. Зафиксировать общую структуру алгоритма на уровне родительского класса, создав соответствующие абстрактные методы-этапы алгоритма. **Функция ``Eat()`` перестает быть виртуальной!**
2. Весь повторяющийся код реализовать на уровне родительского класса.
3. Весь специфический для конкретного фрукта код перенести в реализации методов-этапов на уровне унаследованных классов.


### реализация:
1. Унифицированный метод Eat
```cpp
void Eat() {
        // 1. Вывести название фрукта
        PrintType();
        cout << " : ";

        // 2. Определить качество фрукта
        DetectGoodOrNot();
        cout << " : ";

        // 3. Удалить кожуру
        Peel();
        cout << " : ";

        // 4. Способ поедания (реализуется в подклассах)
        EatManner();

        cout << endl;
    }
```
**2. В private класса Fruit добавили:**
```cpp
void DetectGoodOrNot(){
    if(IsGood())
    {
        cout << "Eating GOOD fruit... ";
    }
    else
    {
        cout << "Eating BAD fruit... ";
    }
}
```    
**3. Реализация для конкретных фруктов**
```cpp    
class Apple : public Fruit // Класс-наследник "Яблоко"
{
public:
    Apple();
    ~Apple() {}

    void PrintType() { cout << "Apple"; }
    void Peel() { cout << "Peel using special tool"; }
};

class Kiwi : public Fruit // Класс-наследник "Киви"
{
public:
    Kiwi() : Fruit(FruitColor::Green) { SetEatingManner(CreateEatingStrategy(EatingMannerEnum::Swallow)); }
    ~Kiwi() {}

    void PrintType() { cout << "Kiwi"; }
    void Peel() { cout << "Peel using small knife"; }
};

class Orange : public Fruit // Класс-наследник "Апельсин"
{
public:
    Orange() : Fruit(FruitColor::Orange) { SetEatingManner(CreateEatingStrategy(EatingMannerEnum::Chop)); }
    ~Orange() {}

    void PrintType() { cout << "Orange"; }
    void Peel() { cout << "Peel using chief knife"; }
};
```

### Краткая характеристика:

Паттерн "Шаблонный метод" задает "скелет" алгоритма в методе, оставляя определение реализации некоторых шагов унаследованным классам. Эти классы могут переопределять некоторые части алгоритма без изменения его структуры.





## Технология uml
- если хочется нарисовать диагрумму классов 
- PlantUML - есть сайтик и приложение на java - специальный язык разметки, позволяющий рисовать разные виды диаграмм

### **Код PlantUML**
```plantuml
@startuml
class Fruit {
  +PrintType()
  +Peel()
  +Eat()
  -DofaulisingMammal()
}

class Apple {
  +PrintType()
  +Peel()
}

class Kiwi {
  +PrintType()
  +Peel()
}

class Orange {
  +PrintType()
  +Peel()
}

Fruit <|-- Apple
Fruit <|-- Kiwi
Fruit <|-- Orange
@enduml
```

---

### **Описание схемы**
1. **Базовый класс `Fruit`:**
   - **Публичные методы:**
     - `PrintType()`: выводит тип фрукта.
     - `Peel()`: очищает фрукт.
     - `Eat()`: реализует процесс употребления фрукта.
   - **Приватный метод:**
     - `DofaulisingMammal()`: внутренняя логика обработки (название требует уточнения).

2. **Производные классы:**
   - **`Apple`, `Kiwi` и  `Orange`:**
     - Наследуют все публичные методы от `Fruit`.
     - Переопределяют методы `PrintType()` и `Peel()` для реализации специфичного поведения (например, разный способ очистки для яблока и киви).

3. **Связи:**
   - Стрелки `◁|--` обозначают наследование: `Apple`, `Kiwi`, `Orange` являются подклассами `Fruit`.

---
Оригиналы файлов в: https://github.com/da-khorkov/FruitFactory-2025/tree/master




