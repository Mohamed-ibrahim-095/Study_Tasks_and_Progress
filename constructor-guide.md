# دليل Constructor في JavaScript

## نظرة عامة

الـ `constructor` هو طريقة خاصة في الـ class تُستخدم لإنشاء وتهيئة نسخة كائن من تلك الـ class.

ملاحظة: هذه الصفحة تقدم شرحاً لصيغة الـ constructor. للاطلاع على خاصية الـ constructor الموجودة في جميع الكائنات، راجع `Object.prototype.constructor`.

## الصيغة البرمجية

```js
constructor() { /* … */ }
constructor(argument0) { /* … */ }
constructor(argument0, argument1) { /* … */ }
constructor(argument0, argument1, /* …, */ argumentN) { /* … */ }
```

### قيود الصيغة البرمجية

هناك بعض القيود على استخدام الـ constructor:
- لا يمكن أن تكون طريقة الـ class المسماة constructor من نوع getter أو setter أو async أو generator
- لا يمكن أن تحتوي الـ class على أكثر من constructor واحد

## الوصف التفصيلي

يتيح الـ constructor إمكانية إجراء أي تهيئة مخصصة يجب تنفيذها قبل استدعاء أي طرق أخرى على الكائن المُنشأ.

### مثال أساسي

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  introduce() {
    console.log(`Hello, my name is ${this.name}`);
  }
}

const otto = new Person("Otto");
otto.introduce(); // Hello, my name is Otto
```

### الـ Constructor الافتراضي

إذا لم تقم بتوفير constructor خاص بك، سيتم توفير constructor افتراضي:

- للـ class الأساسية (base class)، يكون الـ constructor الافتراضي فارغاً:
```js
constructor() {}
```

- للـ class المشتقة (derived class)، يقوم الـ constructor الافتراضي باستدعاء constructor الـ class الأب:
```js
constructor(...args) {
  super(...args);
}
```

### مثال على التعامل مع الأخطاء

```js
class ValidationError extends Error {
  printCustomerMessage() {
    return `فشل التحقق :-( (التفاصيل: ${this.message})`;
  }
}

try {
  throw new ValidationError("رقم هاتف غير صالح");
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(error.name); // سيظهر "Error" وليس "ValidationError"!
    console.log(error.printCustomerMessage());
  } else {
    console.log("خطأ غير معروف", error);
    throw error;
  }
}
```

في المثال السابق، لا يحتاج الـ ValidationError إلى constructor صريح لأنه لا يحتاج إلى أي تهيئة مخصصة. الـ constructor الافتراضي يقوم بتهيئة الـ class الأب Error باستخدام المعامل المُقدم.

### استخدام Constructor مخصص مع Class مشتقة

```js
class ValidationError extends Error {
  constructor(message) {
    super(message); // استدعاء constructor الـ class الأب
    this.name = "ValidationError";
    this.code = "42";
  }

  printCustomerMessage() {
    return `فشل التحقق :-( (التفاصيل: ${this.message}, الرمز: ${this.code})`;
  }
}

try {
  throw new ValidationError("رقم هاتف غير صالح");
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(error.name); // الآن سيظهر "ValidationError"!
    console.log(error.printCustomerMessage());
  } else {
    console.log("خطأ غير معروف", error);
    throw error;
  }
}
```

## خطوات إنشاء كائن جديد

عند استخدام `new` مع class، تتم العملية وفق الخطوات التالية:

1. (في حالة الـ class المشتقة) يتم تنفيذ جسم الـ constructor قبل استدعاء `super()`. هذا الجزء لا يجب أن يصل إلى `this` لأنه لم يتم تهيئته بعد.
2. يتم تنفيذ استدعاء `super()`، والذي يقوم بتهيئة الـ class الأب.
3. يتم تهيئة حقول الـ class الحالية.
4. يتم تنفيذ باقي جسم الـ constructor.

### مثال على الحقول الخاصة

```js
new (class C extends class B {
  constructor() {
    console.log(this.foo());
  }
} {
  #a = 1;
  foo() {
    return this.#a; // خطأ: لا يمكن قراءة العضو الخاص #a
    // السبب الحقيقي ليس أن الـ class لم يُعلن عنه،
    // ولكن لأن الحقل الخاص لم يتم تهيئته بعد
    // عندما يتم تشغيل constructor الـ class الأب
  }
})();
```

## القيم المُرجعة من Constructor

- يمكن للـ constructor في الـ class الأساسية إرجاع أي قيمة
- يجب على الـ constructor في الـ class المشتقة إرجاع كائن أو undefined فقط

```js
class ParentClass {
  constructor() {
    return 1;
  }
}

console.log(new ParentClass()); // ParentClass {}
// يتم تجاهل القيمة المُرجعة لأنها ليست كائناً

class ChildClass extends ParentClass {
  constructor() {
    return 1;
  }
}

console.log(new ChildClass()); // TypeError: يمكن للـ constructor المشتق إرجاع كائن أو undefined فقط
```

## معاملات Constructor

يتبع الـ constructor صيغة الطريقة العادية، لذا يمكن استخدام القيم الافتراضية للمعاملات والمعاملات المتبقية وغيرها:

```js
class Person {
  constructor(name = "مجهول") {
    this.name = name;
  }
  introduce() {
    console.log(`مرحباً، اسمي ${this.name}`);
  }
}

const person = new Person();
person.introduce(); // مرحباً، اسمي مجهول
```

## قيود إضافية

- يجب أن يكون اسم الـ constructor حرفياً. لا يمكن استخدام الخصائص المحسوبة كـ constructor:

```js
class Foo {
  // هذه خاصية محسوبة. لن يتم اعتبارها كـ constructor
  ["constructor"]() {
    console.log("تم الاستدعاء");
    this.a = 1;
  }
}

const foo = new Foo(); // لا يوجد سجل
console.log(foo); // Foo {}
foo.constructor(); // يسجل "تم الاستدعاء"
console.log(foo); // Foo { a: 1 }
```

## أمثلة متقدمة

### استخدام Constructor في Class مشتقة

```js
class Square extends Polygon {
  constructor(length) {
    // هنا، يتم استدعاء constructor الـ class الأب مع الأطوال
    // المقدمة لعرض وارتفاع المضلع
    super(length, length);
    // ملاحظة: في الـ classes المشتقة، يجب استدعاء `super()`
    // قبل استخدام `this`. عدم القيام بذلك سيؤدي إلى خطأ
    this.name = "Square";
  }

  get area() {
    return this.height * this.width;
  }

  set area(value) {
    this.height = value ** 0.5;
    this.width = value ** 0.5;
  }
}
```

### استدعاء super في Constructor مرتبط بنموذج أولي مختلف

```js
class Polygon {
  constructor() {
    this.name = "Polygon";
  }
}

class Rectangle {
  constructor() {
    this.name = "Rectangle";
  }
}

class Square extends Polygon {
  constructor() {
    super();
  }
}

// جعل Square يرث من Rectangle بدلاً من Polygon
Object.setPrototypeOf(Square, Rectangle);

const newInstance = new Square();

// newInstance لا يزال نسخة من Polygon، لأننا لم
// نغير النموذج الأولي لـ Square.prototype
console.log(newInstance instanceof Polygon); // true
console.log(newInstance instanceof Rectangle); // false

// ولكن، لأن super() يستدعي Rectangle كـ constructor،
// فإن خاصية name في newInstance يتم تهيئتها بمنطق Rectangle
console.log(newInstance.name); // Rectangle
```