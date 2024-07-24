## [Dynamic dispatch](https://alschwalm.com/blog/static/2017/03/07/exploring-dynamic-dispatch-in-rust/)

Динамическая диспатчеризация - это механизм, который позволяет вызывать методы для [[Dynamically sized types]] в процессе run-time. (в особенности для trait-объектов)


``` Rust
trait Draw {
    fn draw(&self);
}

struct Circle {
    radius: f64,
}

impl Draw for Circle {
    fn draw(&self) {
        println!("Drawing a circle with radius {}", self.radius);
    }
}

struct Square {
    side: f64,
}

impl Draw for Square {
    fn draw(&self) {
        println!("Drawing a square with side {}", self.side);
    }
}


fn main() {
    let circle = Circle { radius: 5.0 };
    let square = Square { side: 10.0 };

    let shapes: Vec<Box<dyn Draw>> = vec![
        Box::new(circle),
        Box::new(square),
    ];

    for shape in shapes {
        shape.draw();
    }
}
```


`dyn Draw` - это trait-объект (тип), который указывает на любой объект, реализующий trait `Draw`.

---

### [Представление trait-объекта в памяти: ](https://alexeden.github.io/learning-rust/programming_rust/11_traits_and_generics.html#:~:text=In%20memory%2C%20a%20trait%20object,called%20a%20virtual%20table%20(vtable))


- Trait object представлен в виде двух указателей:

	- data pointer. указатель на данные объекта, реализующего trait
		
	- указатель на vtable (vptr), которая содержит указатели на реализации методов для конкретного типа, реализующего данный trait
	
``` Rust
pub struct TraitObject {
	pub data: *mut (),
	pub vtable: *mut (),
}
```

![[trait_obj.png|400]]

---

## Vtables


Vtable (Virtual Method Table) — это структура данных, используемая в языках программирования для поддержки полиморфизма и динамической диспетчеризации. 


Vtable содержит указатели на реализации методов для конкретного типа.

- Это позволяет вызывать методы во время выполнения программы, основываясь на фактическом типе объекта.

---
### Свойства и принцип работы

- `vtables` создаются на этапе компиляции [в сегменте](https://alschwalm.com/blog/static/2017/01/24/reversing-c-virtual-functions-part-2-2/) `data`
	- при создании нового типа, который реализует данный trait  / DST, компилятор создаёт `vtable` для этого trait-объекта.

- в процессе runtime при вызове метода происходит поиск правильной реализации метода для данного типа

- после этого происходит динамический вызов подходящего метода


рассмотрим пример, приведённый выше:

``` Rust
impl Draw for Circle {
    fn draw(&self) {
        println!("Drawing a circle with radius {}", self.radius);
    }
}

fn main() {
    let circle: Box<dyn Draw> = Box::new(Circle { radius: 3.0 });
    let square: Box<dyn Draw> = Box::new(Square { side: 4.0 });

    let shapes: Vec<Box<dyn Draw>> = vec![circle, square];

    for shape in shapes {
        shape.draw();
    }
}
```


- Компилятор создаёт `vtable` для типа `Circle`, которая содержит указатель на метод `draw` для `Circle`.

- Компилятор создает `vtable` для типа `Square`, которая содержит указатель на метод `draw` для `Square`.

- При создании `Box<dyn Draw>` компилятор создаёт структуру, состоящую из двух указателей, один из которых указывает на объект данного типа, а второй - на соответствующую `vtable`.

- При вызове `shape.draw()` компилятор использует указатель на соответствующую `vtable` для поиска *правильного* метода `draw`, а затем вызовет его. 