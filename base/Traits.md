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

## Deref && Drop traits

Deref и Drop - это 2 важных трейта для работы с умными указателями.

### [Deref trait](https://doc.rust-lang.org/std/ops/trait.Deref.html)

*трейт Deref позволяет типам, реализующим данный трейт перегружать* 
*оператор разыменования `*`*

``` Rust
pub trait Deref {
    type Target: ?Sized;

    // Required method
    fn deref(&self) -> &Self::Target;
}
```

- `type Target` — это ассоциированный тип, который указывает на тип, к которому мы хотим получить доступ при разыменовании.


Типы из стандартной библиотеки Rust, реализующие данный трейт:
- `Box<T>`
- `Rc<T>`
- `Arc<T>`
- slices (`&[T]` && `&str`) 
- `Vec<T>`
- `RefCell<T>`
- `Cow<T>`


``` Rust
use std::ops::Deref;

struct MyPointer<T> {
    value: T,
}

impl<T> Deref for MyPointer<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.value
    }
}

fn main() {
    let x = MyPointer { value: "Hello, world!" };
    println!("{}", *x); // Output: "Hello, world!"
}
```

- В неизменяемых контекстах, `*x` (где `T` не является ни ссылкой, ни необработанным указателем) эквивалентно `*Deref::deref(&x)`.


### Automatic dereferencing

Если указанный выше тип реализует трейт `Deref`, то для него актуально автоматическое разыменование (automatic dereferencing).

Это означает, что Rust будет автоматически вызывать метод `deref` для получения ссылки на целевой тип, когда это необходимо.

Если же разработчику нужно нетривиальное разыменование для пользовательского типа,который не представлен в стандартной библиотеке ,то он сам прописывает условие через ассоциированный тип и функцию `deref`.

Вот как это работает:

1. **Ассоциированный тип `Target`**:
    
    - определяется ассоциированный тип `Target` в реализации трейта `Deref`.
	    - `Target` указывает, к какому типу необходимо получить доступ при разыменовании.
        
2. **Функция `deref`**:
    
    - Определяется функция `deref`, которая возвращает ссылку на ассоциированный тип `Target`.
	    - Этот метод определяет, как именно происходит разыменование.


### [Drop trait](https://doc.rust-lang.org/std/ops/trait.Drop.html)

Трейт Drop представлен в языке Rust для очистки памяти, занимаемой объектом, когда он выходит из области видимости.

``` Rust
pub trait Drop {
    // Required method
    fn drop(&mut self);
}
```

Вот как это работает на практике:

1. Когда объект выходит из области видимости, Rust сначала проверяет, реализует ли тип данного объекта трейт `Drop`.

2. Если трейт `Drop` реализован, вызывается метод `drop(&mut self)`.

3. После вызова `drop`, Rust генерирует "drop glue" для объекта, который затем рекурсивно вызывает деструкторы для всех полей этого объекта.

4. Этот процесс продолжается до тех пор, пока не будут вызваны деструкторы для всех полей, гарантируя, что все ресурсы будут освобождены.


"Drop glue" — это механизм, который гарантирует, что все поля типа будут корректно удалены, когда экземпляр типа выходит из области видимости.

"Drop glue" не требует явного определения программистом, так как он генерируется компилятором Rust.

``` Rust
struct MyStruct {
    data: Vec<i32>,
}

impl Drop for MyStruct {
    fn drop(&mut self) {
        println!("Dropping MyStruct with data: {:?}", self.data);
    }
}

fn main() {
    let my_struct = MyStruct { data: vec![1, 2, 3] };
}
```

В этом примере, когда `my_struct` выходит из области видимости, метод `drop` для `MyStruct` будет вызван автоматически, и выведется сообщение о том, что `MyStruct` был удален. 

- Компилятор Rust автоматически добавит код для вызова `drop`
   для поля `data`, чтобы вектор `Vec<i32>` также был корректно удален.

