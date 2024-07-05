**Трейт (Trait):** - это абстрактный интерфейс, который определяет набор методов, которые могут быть реализованы типами, которые реализуют данный трейт.

Трейты частично реализуют ***полиморфизм***,за счёт того, что их реализацию можно определить для произвольного типа.


Trait-объекты - это способ реализации абстрактных типов данных

- использование trait-объектов позволяет группировать разные типы данных, организованные по каким-либо признакам и свойствам.

- ***Инкапсуляция*** с помощью Trait-объектов позволяет скрыть детали реализации от пользователя.

- ***Взаимозаменяемость Trait-объектов*** - это когда один и тот же код может работать с различными типами данных, реализующими определенный Trait.

- Трейты являются альтернативой интерфейсам в других языках, но обладают большей гибкостью.

---

## Реализация трейтов

- Трейты объявляются с помощью ключевого слова `trait`.

- Внутри трейта можно объявлять методы и ассоциированные типы

- Трейты могут иметь определение методов по умолчанию
	- тип, для которого будет реализован трейт по умолчанию будет иметь метод, определённый по умолчанию.


``` Rust

trait Summary {
    fn summarize(&self) -> String;

    fn summarize_author(&self) -> String {
        String::from("Unknown author")
    }
}

struct Article {
    title: String,
    author: String,
    content: String,
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("Article: {}, by {}", self.title, self.author)
    }

    fn summarize_author(&self) -> String {
        format!("Author: {}", self.author)
    }
}
```



---

## Dynamic dispatch

Динамическая диспатчеризация - это механизм, который позволяет вызывать методы для полиморфных типов в процессе run-time.

Это контрастирует со Статической диспатчеризацией, достигаемой с использованием generics<>, где тип $T$ необходимо знать во время компиляции для создания метода / функции с конкретным вызываемым шаблонным параметром.


``` Rust
trait Walkable {
	fn walk(&self);
}

struct Cat;
struct Dog;

impl Walkable for Cat {
	fn walk(&self) {
	    println!("Cat is walking");
    }
}

impl Walkable for Dog {
	fn walk(&self) {
	    println!("Dog is walking");
	}
}

fn dynamic_dispatch(w: &dyn Walkable) {
	w.walk();
}


fn main() {
	let cat = Cat{};
	let dog = Dog{};
	
	dynamic_dispatch(&cat);
	dynamic_dispatch(&dog);
}
```


функция `dynamic_dispatch` принимает ссылку на любой тип, реализующий `Walkable`.



- Динамический подход обычно медленнее, чем статический из-за необходимости поиска соответствующей функции в таблице виртуальных функций (vtable) во время выполнения.

- Динамический подход эффективнее использует память, так как не требует создания множественных копий функции для различных типов.


---

## Trait bounds

Трейты можно использовать для ограничения  generic-типов. 

Trait bounds позволяет гарантировать, что generic-тип будет обладать определенными методами.

Ограничение по трейту используется для указания того, что тип должен реализовывать определенный набор поведений.

``` Rust
fn notify<T: Summary>(item: &T) {
    println!("Breaking news: {}", item.summarize());
}
```

*В данном случае аргумент функции T будет ограничен и может быть только одним из типов, для которых реализован трейт `Summary`.* 


при этом ограничения можно увеличивать:
``` Rust
fn notify<T: Summary + Display>(item: &T) {
    println!("Breaking news: {}, {}", item.summarize(), item);
}
```

В этом случае функция `notify` принимает ссылку на объект типа $T$, который реализует трейты `Summary` и `Display`.


$\triangle$ если же ограничений много, то лучше использовать ключевое слово ***where***
``` Rust
fn notify<T>(item: &T)
where
    T: Summary + Display + Something_else,
{
    println!("Breaking news: {}, {}", item.summarize(), item);
}
```

---

## Generic Traits

Обобщенные трейты позволяют определять трейты, которые могут работать с различными типами данных. Это достигается с помощью generic-параметров.

``` Rust 
trait Container<T> {
    fn new() -> Self;
    fn add(&mut self, item: T);
    fn contains(&self, item: &T) -> bool;
}

struct MyContainer<T> {
    items: Vec<T>,
}

impl<T> Container<T> for MyContainer<T> {
    fn new() -> Self {
        MyContainer { items: Vec::new() }
    }

    fn add(&mut self, item: T) {
        self.items.push(item);
    }

    fn contains(&self, item: &T) -> bool {
        self.items.contains(item)
    }
}
```


---

## Supertraits

Супертрейты - это концепция языка Rust, которая позволяет явно указывать, что трейт, который мы определяем, требует, чтобы типы, реализующие этот трейт, также реализовывали другой трейт. 

Это позволяет трейту зависеть от функциональности другого трейта.

Example:

``` Rust
use std::fmt;

trait PrettyPrint: fmt::Display + fmt::Debug {
    fn pretty_print(&self) {
        println!("Display: {}", self);
        println!("Debug: {:?}", self);
    }
}
```

В этом примере $PrettyPrint$ требует, чтобы типы, реализующие его, также реализовывали трейты $Display$ и $Debug$.

---

## trait 'Sized'

Sized - это trait, который используется для представления типов, размер которых известен на этапе компиляции.

по умолчанию все примитивные типы являются реализующими данный трейт, исключением являются [[Dynamically sized types && Exotically sized types]].

``` Rust
fn print_sized<T: Sized>(value: T) {
    println!("Value: {:?}", value);
}

fn main() {
    let x = 42;
    print_sized(x); // OK, i32 in Sized

    // let slice = &[1, 2, 3];
    // print_sized(slice); // ERROR, &[i32] is not in Sized
}
```


---

